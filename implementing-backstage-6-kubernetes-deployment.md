---
title: "Implementing Backstage: Kubernetes Deployment"
authors:
  - Carlos Eduardo Inocencio Alanis
---
This final part of the "Implementing Backstage" series focuses on how to deploy Backstage on Kubernetes. This tutorial is a direct continuation of Using the Kubernetes Plugin in Backstage, which you should complete before tackling this one.

The other installments in this series covered [getting started](https://www.kosli.com/blog/implementing-backstage-getting-started/), using the core features,  integrating with existing tools using plugins, and security and compliance. If you're looking for an introduction to Backstage, you can read the [first article](https://www.kosli.com/blog/evaluating-backstage-why-backstage/) in the "Evaluating Backstage" series.

There are many reasons why deploying Backstage using Kubernetes is a great idea. The ability to [scale based on demand](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) is probably the biggest one. Depending on your organization's network architecture, having the Postgres database deployed right next to your service in the cluster might simplify its administration. If you also implement a CI/CD pipeline with your deployment, you can have a reliable service that can be updated with virtually zero downtime for the end user.

In this tutorial, you'll learn how to deploy Backstage on Kubernetes using Docker and Terraform.

## Deploying Backstage on Kubernetes

Unlike the previous part of this series, this tutorial only covers cloud deployment. For a local Kubernetes deployment, refer to the [official tutorial](https://backstage.io/docs/deployment/k8s), which shares similar steps to an on-premise deployment. The process of deploying a Kubernetes cluster on the cloud is also outside the scope of this article, but there are [many guides](https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks) on how to do it.

You're going to create the supporting infrastructure to deploy Backstage using Kubernetes. You'll use Docker as a container solution and then learn how to configure a service, load balancer, and cluster using Terraform. All necessary code to use infrastructure as code (IaC) to deploy your app is provided.

### The Container

First, you need a container. Using the [Docker tutorial](https://backstage.io/docs/deployment/docker/) from the official Backstage documentation as a foundation, let's go over the steps to build a Backstage app into a deployable Docker image.

You have two options to create the app: using a single package that contains both the backend and frontend, or deploying them separately. If you're uncertain about whether you need to separate them, it's likely that you don't. There's nothing wrong with serving each separately, but it adds complexity to the code. For this reason, the tutorial keeps both in the same package.

By default, the app comes with the following `Dockerfile` in `packages/backend`:

```yaml
FROM node:16-bullseye-slim

# Install isolate-vm dependencies, which are needed by the @backstage/plugin-scaffolder-backend.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential && \
    yarn config set python /usr/bin/python3

# Install sqlite3 dependencies. You can skip this if you don't use sqlite3 in the image,
# in which case you should also move better-sqlite3 to "devDependencies" in package.json.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev

# From here on, you use the least-privileged `node` user to run the backend.
USER node

# This should create the app dir as `node`.
# If it is instead created as `root`, then the `tar` command below will fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`) so the app dir is correctly created as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV production

# Copy the repo skeleton first to avoid unnecessary Docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files you might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
```

This code is perfectly serviceable, but if you're deploying an application in the real world, there are a few changes that can help you achieve better results.

First, use `node:18` instead of `16`. Node 18 is now (as of August 2023) the LTS version, so there is really no point in keeping `node:16` here:

```
FROM node:18-bullseye-slim
```

If you want to use [TechDocs](https://backstage.io/docs/features/techdocs/), you'll need to install them:

```shell
RUN apt-get update && apt-get install -y python3-pip git
RUN pip3 install mkdocs-techdocs-core==1.1.7
```

This might be an edge case, but if the app is not using the right value for `baseUrl`, you can set an override right before running the `CMD` instruction by including these lines of code:

```shell
ENV APP_CONFIG_app_baseUrl "http://${APP_HOST}"
ENV APP_CONFIG_backend_baseUrl "http://${APP_HOST}"
ENV APP_CONFIG_auth_environment "production"
```

You can have [several configuration files](https://backstage.io/docs/conf/writing/) and use them to separate the configuration that's relevant to the environment you are deploying to. Here's an example of an `app-config.production.yaml` file that contains configuration only used for the production deployment:

```yaml
app:
  # Should be the same as backend.baseUrl when using the `app-backend` plugin.
  baseUrl: http://${APP_HOST}

backend:
  # Note that the baseUrl should be the URL that the browser and other clients
  # use when communicating with the backend. In other words, it needs to be
  # reachable not just from within the backend host, but from all of your
  # callers. When its value is "http://localhost:7007", it's strictly private
  # and can't be reached by others.
  baseUrl: http://${APP_HOST}
  # The listener can also be expressed as a single <host>:<port> string. In this case, you bind to
  # all interfaces, the most permissive setting. The right value depends on your specific deployment.
  listen: ':${APP_PORT}'

  # Config options: https://node-postgres.com/apis/client
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      # https://node-postgres.com/features/ssl
      # You can set the sslmode configuration option via the `PGSSLMODE` environment variable
      # See https://www.postgresql.org/docs/current/libpq-ssl.html Table 34.1. SSL Mode Descriptions (such as `require`)
      # ssl:
      #   ca: # If you have a CA file and want to verify it, you can uncomment this section
      #     $file: <file-path>/ca/server.crt
auth:
  # You don't need this if you own the domain where you are deploying the app,
  # but if you are using host files and something like a reverse proxy with separate frontend and backend,
  # there could be a disconnect between the baseUrl and the URL displayed in the browser.
  # This can help authorize alternative frontend hosts:
  # experimentalExtraAllowedOrigins:
    # - http://${APP_HOST} # This could be used, for example, if your baseUrl contains a custom port but the URL in the browser does not
  # See https://backstage.io/docs/auth/ to learn about auth providers
  
  # You can have different configs for different environments. Here is the client when deploying to "production". This article does not cover authentication.
  environment: production
  providers: 
    gitProvider:
      production:
        clientId: ${gitProvider_APP_CLIENT_ID}
        clientSecret: ${gitProvider_APP_CLIENT_SECRET}  

# You can have an override of the catalog that only happens when deploying to production. Here is an example of templates living in a Git repository. This article does not cover custom templates.
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]
  locations:
    - type: url
      target: https://gitProvider.com/someOrg/someNameSpace/-/blob/main/all-templates.yaml
      rules:
        - allow: [Template]
```

If you are using an additional YAML configuration file like the one shown before, make sure to include it in the `CMD` instruction:

```yaml
CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
```

Here is what the resulting `Dockerfile` looks like for an actual deployment with all the changes discussed:

```yaml
# This Dockerfile builds an image for the backend package.
# It should be executed with the root of the repo as docker context.
#
# Before building this image, be sure to run the following commands in the repo root:
#
# yarn install
# yarn tsc
# yarn build:backend
#
# Once the commands have been run, you can build the image using `yarn build-image`

# Use Node 18 instead of Node 16
FROM node:18-bullseye-slim

# Install isolate-vm dependencies, which are needed by the @backstage/plugin-scaffolder-backend.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential && \
    yarn config set python /usr/bin/python3

# Install sqlite3 dependencies. You can skip this if you don't use sqlite3 in the image,
# in which case you should also move better-sqlite3 to "devDependencies" in package.json.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev

# Install TechDocs
RUN apt-get update && apt-get install -y python3-pip git
RUN pip3 install mkdocs-techdocs-core==1.1.7

# From here on, you use the least-privileged `node` user to run the backend.
USER node

# This should create the app dir as `node`.
# If it is instead created as `root`, then the `tar` command below will fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`) so the app dir is correctly created as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV production

# These are merely examples of arguments passed at build time. Your deployment might use more or not need them at all.
ARG APP_HOST
ARG APP_PORT
ARG POSTGRES_HOST
ARG POSTGRES_PORT
ARG POSTGRES_USER
ARG POSTGRES_PASSWORD
ARG gitProvider_APP_CLIENT_ID
ARG gitProvider_APP_CLIENT_SECRET

# These should be taken from a secure location during the CI/CD pipeline and put in Kubernetes secrets, never hard-coded.
ENV APP_HOST=${APP_HOST}
ENV APP_PORT=${APP_PORT}
ENV POSTGRES_HOST=${POSTGRES_HOST}
ENV POSTGRES_PORT=${POSTGRES_PORT}
ENV POSTGRES_USER=${POSTGRES_USER}
ENV POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
ENV gitProvider_APP_CLIENT_ID=${gitProvider_APP_CLIENT_ID}
ENV gitProvider_APP_CLIENT_SECRET=${gitProvider_APP_CLIENT_SECRET}

# Copy the repo skeleton first to avoid unnecessary Docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files you might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Set if you are facing issues with the app selecting the right host when deployed.
ENV APP_CONFIG_app_baseUrl "http://${APP_HOST}"
ENV APP_CONFIG_backend_baseUrl "http://${APP_HOST}"
ENV APP_CONFIG_auth_environment "production"
ENV NODE_OPTIONS "--max-old-space-size=1536" # Highly dependent on your deployment; see https://nodejs.org/api/cli.html#--max-old-space-sizesize-in-megabytes

CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
```

This `Dockerfile` will create a production-ready image as long as you are doing a host build. It's faster than the alternative multi-stage build, which you can read about [here](https://backstage.io/docs/deployment/docker/#multi-stage-build).

You can now create your image by going to the root of your project and running:

```
docker image build . -f packages/backend/Dockerfile --tag backstage --build-arg SOME_ARG=${SOME_VALUE}
```

### The Deployment

This tutorial uses Terraform for the deployment, as it's the most cloud-agnostic approach to infrastructure as code. The example covered will be for AWS, but you can adapt the concepts to any cloud provider.

#### Configuring Providers

To connect to an existing Kubernetes cluster deployment using Terraform, you first need to configure providers. Here is the exact minimum config to achieve this:

```hcl
provider "aws" {    
  region = "us-east-1"    
}    
  
provider "kubernetes" {    
  host                   = var.EKS_ENDPOINT_URL    
  cluster_ca_certificate = base64decode(var.AWS_EKS_CA_DATA)    
    exec {    
      api_version = "client.authentication.k8s.io/v1beta1"    
      args        = ["eks", "get-token", "--cluster-name", var.cluster_name]    
      command     = "aws"    
  }    
}  
```

Configuring the providers is an essential step. Both `EKS_ENDPOINT_URL` and `AWS_EKS_CA_DATA` can be obtained from the AWS console.

#### Configuring Kubernetes Secrets

Next, your application might need access to some sensitive data, like GitHub tokens or user passwords. If that is the case, you need to configure Kubernetes secrets. Please do not hard-code secrets or leave them as plain environment variables. Notice how the Postgres data is collected directly from the source without ever being exposed anywhere:

```hcl
resource "kubernetes_secret" "backstage_secrets" {      
  metadata {    
    name      = "backstage-secrets"      
    namespace = "backstage"      
  }     
     
  data = {      
    POSTGRES_HOST            = aws_rds_cluster.aurora_postgres.endpoint      
    POSTGRES_USER            = aws_rds_cluster.aurora_postgres.master_username      
    POSTGRES_PASSWORD        = aws_rds_cluster.aurora_postgres.master_password      
    SOME_SECRET  = var.SOME_SECRET
    SOME_OTHER_SECRET  = var.SOME_OTHER_SECRET
  }     
}  
```

#### Deploying the Infrastructure

You're now ready to deploy your infrastructure. As mentioned, you need a load balancer, which coordinates the requests between the user and the pods running the Backstage app. In Kubernetes, an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) manages the exposure of HTTP endpoints to services running inside the cluster. In [AWS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html), the application load balancer handles this function natively. You don't need to configure a separate ingress and external load balancer.

With Terraform, you only need to deploy a `kubernetes_ingress_v1` resource for AWS to provision the necessary infrastructure. Use the following code to deploy this resource, which you will then create in the next step:

```hcl
resource "kubernetes_ingress_v1" "application_load_balancer" {  
  metadata {  
    name      = "application-load-balancer"  
    namespace = "backstage"  
    annotations = {  
      # These annotations indicate that it's an internal application load balancer; ip (https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#target-type) is needed for sticky sessions
      "kubernetes.io/ingress.class" = "alb"  
      "alb.ingress.kubernetes.io/scheme" = "internal"  
      "alb.ingress.kubernetes.io/target-type" = "ip"
    }  
  }  
  
  # The spec is set up so it redirects everything targeted at the APP_HOST/* domain to the pod, no matter the path selected
  spec {  
    rule {  
      host = var.APP_HOST
      http {  
        path {  
          path = "/"  
          path_type = "Prefix"
          backend {  
            service {  
              name = kubernetes_service_v1.backstage.metadata[0].name  
              port {  
                number = kubernetes_service_v1.backstage.spec[0].port[0].port  
              }  
            }  
          }  
        }  
      }  
    }  
  }  
}  
```

This ingress is pointing all calls to `kubernetes_service_v1`, which is responsible for creating pods as needed. By linking the ingress resource to the service, every new pod created in the cluster by the service will be automatically routed to the load balancer. Use the following code to create `kubernetes_service_v1`:

```hcl
# The service will register the pods using the selector
resource "kubernetes_service_v1" "backstage" {  
  metadata {  
    name      = "backstage"  
    namespace = "backstage"  
    labels = {  
      app = "backstage"
      "backstage.io/kubernetes-id" = "backstage-app"
    }  
  }  
  
  spec {  
    selector = {  
      app = "backstage"
      "backstage.io/kubernetes-id" = "backstage-app" 
    }  
  
    port {  
      port        = var.APP_PORT  
      target_port = var.APP_PORT  
    }  
    type = "NodePort"
  }  
}  

# The deployment will take care of defining the configuration of the pods launched
resource "kubernetes_deployment" "backstage" {  
  metadata {  
    name      = "backstage"  
    namespace = "backstage"  
    labels = {  
      app = "backstage"
      "backstage.io/kubernetes-id" = "backstage-app" 
    }  
  }  
  
  # Set the number of replicas (this uses just one for dev purposes)
  spec {  
    replicas = 1  
   
    # Remember that the annotations help the service pick up the pods, but the backstage.io/ annotation is for the component plugin in Backstage
    selector {  
      match_labels = {  
        app = "backstage" 
        "backstage.io/kubernetes-id" = "backstage-app" 
      }  
    }  
  
    template {  
      metadata {  
        labels = {  
          app = "backstage"  
          "backstage.io/kubernetes-id" = "backstage-app"
        }  
      }  
  
      # Define where the image for the app is stored (this uses ECR)
      spec {  
        container {  
          image = "${var.BACKSTAGE_ECR}:${var.COMMIT_SHA}"
          image_pull_policy = "Always"
          name  = "backstage"  
          port {  
            container_port = var.APP_PORT  
          }  
          # Set secrets as env variables in a secure way
          env_from {  
            secret_ref {  
              name = kubernetes_secret.backstage_secrets.metadata[0].name  
            }  
          }  
          # Set the non-sensitive env vars
          env {  
            name  = "APP_HOST"  
            value = var.APP_HOST  
          }
          env {  
            name  = "APP_PORT"  
            value = var.APP_PORT  
          }
          env {  
            name  = "POSTGRES_PORT"  
            value = aws_rds_cluster.aurora_postgres.port  # "5432"  
          }  
          env {  
            name  = "POSTGRES_DB"  
            value = "backstage"  
          }
          # Set the resources allocated to the pod (you can use whatever you want)
          resources {  
            limits = {  
              cpu    = "4"  
              memory = "8Gi"  
            }  
            requests = {  
              cpu    = "4"  
              memory = "8Gi"  
            }  
          }  
        }  
      }  
    }  
  }  
}  
```

This code ensures there is always one live replica of your app. Notice how the SHA hash of the code is used to select the container image. Tagging each new container image with the SHA hash allows you to use a CI/CD pipeline to always pick up the latest code and deploy it to the cluster with no disruption to the user.

The flow is as follows:

1. You push new code
2. A CI/CD pipeline creates a new container image with the new code and tags it using the SHA hash
3. That same variable is then used in the Terraform code to tell the service which image it needs to use
4. The CI/CD pipeline applies the Terraform changes, making your Kubernetes service launch a new container and delete the old one

With that flow set up, you now need to make sure there is proper IAM authorization for the resources to communicate with each other. Use the following code to allow the resource to use the [load balancer](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html):

```hcl
# Allow the role to be assumed by EKS
resource "aws_iam_role" "aws_load_balancer_controller" {  
  name = "aws-load-balancer-controller" 
 
  assume_role_policy = jsonencode({  
    Version = "2012-10-17"  
    Statement = [  
      {  
        Action = "sts:AssumeRole"  
        Effect = "Allow"  
        Principal = {  
          Service = "eks.amazonaws.com"  
        }  
      }  
    ]  
  })  
} 
 
resource "aws_iam_role_policy_attachment" "aws_load_balancer_controller_policy_attachment" {  
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"  
  role       = aws_iam_role.aws_load_balancer_controller.name  
} 
 
resource "aws_iam_role_policy_attachment" "aws_load_balancer_controller_vpc_policy_attachment" {  
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSVPCResourceController"  
  role       = aws_iam_role.aws_load_balancer_controller.name  
} 

resource "aws_iam_role_policy" "aws_load_balancer_controller_additional_policy" {  
  name = "aws-load-balancer-controller-additional"  
  role = aws_iam_role.aws_load_balancer_controller.id 
  # You can get this policy from https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
  policy = file("${path.module}/assets/controller-policy.json")
}  
```

This will allow your resources to properly communicate with the AWS APIs.

This final step will vary depending on your deployment. Backstage uses a Postgres database to keep track of the state of the catalog and resources, and there are a couple of options to deploy it. First, you could deploy Backstage's database in Kubernetes, which would make it easier to access because it would live in the same cluster as the rest of the app. One disadvantage to this approach would be having to administrate the database infrastructure.

Keeping with the cloud-focused approach, this tutorial instead creates a separate managed database using [Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraPostgreSQL.html). You can use the following Terraform code to do that:

```hcl
# Create a new RDS subnet group for the Aurora PostgreSQL database
resource "aws_db_subnet_group" "aurora_postgres_subnet_group" {  
  name       = "aurora-postgres-subnet-group"  
  subnet_ids = var.subnet_ids
  
  tags = {  
    Name = "Aurora PostgreSQL Subnet Group"  
  }  
}  
  
# Create a security group that allows Kubernetes pods to communicate with the PostgreSQL DB
resource "aws_security_group" "aurora_postgres_sg" {  
  name        = "aurora-postgres-sg"  
  description = "Allow inbound traffic from Kubernetes pods to PostgreSQL DB"  
  vpc_id      = var.vpc_id
  
  ingress {  
    from_port   = 5432  
    to_port     = 5432  
    protocol    = "tcp"  
    cidr_blocks = ["172.23.40.0/24"]  
  }  
}  
# Create a password
resource "random_password" "master"{
  length           = 16
  special          = true
  override_special = "_!%^"
}

resource "aws_secretsmanager_secret" "password" {
  name = "backstage-postgres-password"
}

# Save the password without anyone looking at it
resource "aws_secretsmanager_secret_version" "password" {
  secret_id = aws_secretsmanager_secret.password.id
  secret_string = random_password.master.result
}

# Get the password saved
data "aws_secretsmanager_secret_version" "password" {
  secret_id = aws_secretsmanager_secret.password.id
  depends_on = [
    aws_secretsmanager_secret_version.password
  ]
}

# Create an Aurora PostgreSQL database cluster and use the password created before
resource "aws_rds_cluster" "aurora_postgres" {  
  cluster_identifier      = "backstage-aurora-postgres"  
  engine                  = "aurora-postgresql"  
  db_subnet_group_name    = aws_db_subnet_group.aurora_postgres_subnet_group.name  
  vpc_security_group_ids  = [aws_security_group.aurora_postgres_sg.id]  
  master_username         = "master"  
  master_password         = data.aws_secretsmanager_secret_version.password.secret_string
  backup_retention_period = 7  
  preferred_backup_window = "07:00-09:00"  
  skip_final_snapshot     = true  
}  
  
# Create an instance in the previous cluster
resource "aws_rds_cluster_instance" "aurora_postgres_instance" {  
  identifier         = "backstage-aurora-postgres-instance"  
  cluster_identifier = aws_rds_cluster.aurora_postgres.id  
  engine             = "aurora-postgresql"
  instance_class     = "db.r5.large"  
}  
```

That's it from the Terraform perspective. Just create a file to declare the variables you end up using and modify it to fit your needs. You have a fully working Backstage deployment. It's strongly suggested that you use some sort of CI/CD pipeline to deploy it.

## Conclusion

You’ve now learned how to deploy your Backstage application to a Kubernetes cluster. You used Docker as a container solution, Terraform for the deployment, and AWS EKS as a cloud provider. It's worth noting that you can adapt the concepts and instructions in this tutorial to any solution you like.

That concludes the "Implementing Backstage" series. If you’ve followed along from the start, you should have all the essential tools for a working Backstage implementation. However, implementing the tool is only the first step; the real question is how do you succeed with it? To find out, continue to the next series, "Succeeding with Backstage."


---
title: "Implementing Backstage 5: Implementing Backstage: Kubernetes Plugins"
authors:
  - Carlos Eduardo Inocencio Alanis
---

This second last part of the "Implementing Backstage" series explains how to use the Kubernetes plugin in Backstage using real-world scenarios.

The previous installments covered [getting started](https://www.kosli.com/blog/implementing-backstage-getting-started/), using the core features,  integrating with existing tools using plugins, and security and compliance. If you're entirely new to Backstage and want to learn more, you can read the [first entry](https://www.kosli.com/blog/evaluating-backstage-why-backstage/) in the "Evaluating Backstage" series.

In this article, you'll first learn how to install the official Kubernetes plugin. You'll then learn how to configure the plugin and the catalog components for both local and cloud deployments. At every point in this article, the focus will be on real-world deployments, so all the code necessary will be provided to use infrastructure as code (IaC) to deploy your app.

## Monitoring Kubernetes Resources in Backstage

This article assumes you have some basic familiarity with Backstage and Kubernetes. That being said, if you need help creating a new Backstage app, you can take a look [here](https://www.kosli.com/blog/implementing-backstage-getting-started/). You'll also need a running Backstage app to follow along.

Most articles about using the Backstage Kubernetes plugin focus on monitoring a local deployment. For example, it's very common to see examples covering how to integrate the plugin with a local [minikube](https://minikube.sigs.k8s.io/docs/start/) cluster, which is understandable as very few things can go wrong during its setup. This article takes a slightly different approach, demonstrating both how to deploy locally and how to connect to a cluster running on Amazon EKS. For each step, two code solutions will be provided: one for deploying a local Kubernetes cluster and one that's a more complex "real-world" example explaining the configuration needed to connect to a Kubernetes cluster running on AWS.

Seeing how the code works in real-world implementations will help you understand *why* you need to run certain commands. This should help prevent you from getting lost when dealing with this challenge in a professional setting.

Backstage relies entirely on a [software catalog](https://backstage.io/docs/features/software-catalog/) as the source of truth about every entity. The Kubernetes plugin looks for the catalog definition of each component (a type of entity) and tries to associate it with a pod running on a cluster using [annotations](https://backstage.io/docs/features/kubernetes/configuration/#common-backstageiokubernetes-id-label). To achieve this, it needs to have knowledge of the clusters where the pods could be running, authorization to access these clusters, and a method to connect each component to a specific pod.

### Setting Up the Kubernetes Frontend Plugin

You'll first set up the Kubernetes plugin itself, specifically the frontend. This plugin renders the information about the pod on the catalog page for each entity.

To do this, you'll need a running cluster to monitor. If you don't already have one, [minikube](https://minikube.sigs.k8s.io/docs/start/) is a popular and easy-to-use tool. For its cloud counterpart, you'll be accessing a Kubernetes cluster running on Amazon EKS, which you can set up by following the [AWS documentation](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html).

To install the Kubernetes plugin in your Backstage app, go to the root of your project and run the following command:

```bash
yarn add --cwd packages/app @backstage/plugin-kubernetes
```

This installs the frontend plugin, which you'll need to tell the app to use. The code that renders the frontend app can be found inside `packages/app`. Specifically, each entity is rendered using the code in `packages/app/src/components/catalog/EntityPage.tsx`.

In the `EntityPage` React code, you can modify the elements displayed when viewing details about any entity in the software catalog. You can, for example, add a tab to monitor code vulnerabilities. Find `serviceEntityPage`, which controls what's displayed on the page. You can add a tab to display the Kubernetes information with the following code:

```javascript
import { EntityKubernetesContent } from '@backstage/plugin-kubernetes';
....
const serviceEntityPage = (
  <EntityLayout>
    <EntityLayout.Route path="/kubernetes" title="Kubernetes">
      {/* If you don't configure the refresh interval it defaults to 10 seconds */}
      <EntityKubernetesContent refreshIntervalMs={10000} /> 
    </EntityLayout.Route>
    ....
  </EntityLayout>
);
```

That's all the code changes needed for the frontend to render properly. For the plugin to display real data, you now need to configure the backend.

### Setting Up the Kubernetes Backend Plugin: Cloud and Local

The backend plugin is installed in a very similar way to the frontend. Run the following command at the root of your project:

```bash
yarn add --cwd packages/backend @backstage/plugin-kubernetes-backend
```

After that, you have to modify the backend code of the Backstage app so it uses the new plugin. You have to create a new file that will contain the code for the Kubernetes Backstage plugin. The [official documentation](https://backstage.io/docs/features/kubernetes/installation/#adding-kubernetes-backend-plugin) suggests you call it `packages/backend/src/plugins/kubernetes.ts`, and this article uses that same name for consistency.

Create the file and add this code:

```javascript
import { KubernetesBuilder } from '@backstage/plugin-kubernetes-backend';
import { Router } from 'express';
import { PluginEnvironment } from '../types';
import { CatalogClient } from '@backstage/catalog-client';

export default async function createPlugin(
  env: PluginEnvironment,
): Promise<Router> {
  const catalogApi = new CatalogClient({ discoveryApi: env.discovery });
  const { router } = await KubernetesBuilder.createBuilder({
    logger: env.logger,
    config: env.config,
    catalogApi,
    permissions: env.permissions,
  }).build();
  return router;
}
```

The above file imports everything you need to construct the backend plugin and communicate with the catalog. The [`CatalogClient`](https://backstage.io/docs/reference/catalog-client/) is used to interact with your app's catalog using the plugin environment. It creates an instance of the Kubernetes backend plugin using the `createBuilder` method, which then consumes all the resources you previously imported.

For your app to use this new function, you need to go to `packages/backend/src/index.ts` and modify it. Import the code you just created and add a few lines to the `main` function:

```javascript
// At the beginning of the code
import kubernetes from './plugins/kubernetes'; 
// ...rest of the code
async function main() { 
  // ...other definitions
  const kubernetesEnv = useHotMemoize(module, () => createEnv('kubernetes'));
  apiRouter.use('/kubernetes', await kubernetes(kubernetesEnv));
```

Your application now has a frontend and backend Kubernetes plugin. You should be able to see the Kubernetes tab on any service-type component's page. Don't worry if you don't have a component yet; you'll create one later in this article.

![Example of empty Kubernetes component](https://i.imgur.com/lePGxHe.png)

As you can see, the plugin itself is telling you that more configuration is needed.

### Configuring the Plugin

There are [several ways](https://backstage.io/docs/features/kubernetes/configuration/#clusterlocatormethods) for the Kubernetes plugin to access your cluster. Essentially, they involve the process of authenticating with the cluster, and the specific authentication approach used by the plugin may vary depending on the location of your cluster.

If you are running a minikube cluster, the easiest approach is to use a service account. Every Kubernetes deployment comes with a [default service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#use-the-default-service-account-to-access-the-api-server) you can use. You just need to assign a [ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) to this account.

Going back to the real-world scenario demonstrates one of the shortcomings of Backstage: sometimes there is subpar official documentation. At the time of writing, the official [Backstage Kubernetes config documentation](https://backstage.io/docs/features/kubernetes/authentication/#server-side-providers) does not specify how to configure the plugin with an EKS cluster. As a result, there are few online tutorials on the topic. However, this also showcases one of the biggest strengths of open source tools: [users can help improve them](https://github.com/backstage/backstage/pull/19588#issuecomment-1692393144). As that improvement is yet to be approved, let's go over how to configure it here.

#### Authenticating to the Provider

Now that the plugin backend code is ready, the app needs to be configured (using the `app-config.yaml` file) to locate the Kubernetes cluster. When the cluster is running locally, only one piece of configuration is needed: a block detailing how to connect to the cluster's endpoint. When the cluster is running on a cloud provider, an additional piece of configuration is also needed: a block detailing how to authenticate to the cloud provider where the cluster is running.

To access a cluster on the cloud, your Backstage application has to first authenticate to the provider (in this case, AWS). You can use the official [`integration-aws-node`](https://github.com/backstage/backstage/tree/master/packages/integration-aws-node) module to configure authentication for several AWS accounts. You have two options. The first one is to authenticate using static access keys. This is not recommended for production deployments because long-lived credentials have a higher risk of unintentional exposure. The second option is to use short-lived keys obtained from assuming a role. This type of credential rotates every couple of hours, limiting the risk of unintentional exposure.

If you're using keys from a role to authenticate, you need to create one that can be [assumed by your app](https://docs.aws.amazon.com/eks/latest/userguide/security_iam_service-with-iam.html). Configure the role with the necessary [access to the cluster](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html). Then, export the resulting temporary credentials as environment variables.

If you are targeting a cluster on the cloud, the new configuration block for AWS in `app-config.yaml` should look like this:

```yaml
# Be mindful, this does not go inside the "auth:" block
aws:
  mainAccount:
    roleName: backstage-tutorial-role  
    region: us-east-1
  accounts: # Add any additional accounts here if needed
  accountDefaults: # Add defaults here if needed
```

Remember that if you are only running the cluster locally, no authentication is needed to an external provider (only the endpoint configuration that will be covered in the next section).

In the previous `aws` configuration block, replace `backstage-tutorial-role` with your role's name. If you are running the Kubernetes cluster both on the cloud and in the Backstage app itself, you need to configure the role to allow the compute service (*eg* EC2, EKS) that's hosting the app to assume the role you are configuring.

Alternatively, if you are running the Backstage app locally and understand the risks associated with having long-lived credentials, you can use the quicker approach of static AWS keys to authenticate:

```yaml
aws:
  mainAccount:
  accounts: # Add any additional accounts here if needed
    - accountId: 'XXXXXXXXXXXX'
      accessKeyId: ${AWS_ACCESS_KEY_ID}
      secretAccessKey: ${AWS_SECRET_ACCESS_KEY}
      region: us-east-1
  accountDefaults: # Add defaults here if needed
```

At least one of these configuration blocks is needed to use the AWS [`authProvider`](https://backstage.io/docs/features/kubernetes/configuration/#clustersauthprovider).

#### Configuring Access in Backstage

To configure access to a Kubernetes cluster in Backstage, your app needs to be aware of the cluster and know how to connect to it. To achieve this for AWS, go to the `kubernetes` block in the `app-config.yaml` file and create a new block with the following code:

```yaml
kubernetes:
  serviceLocatorMethod:
    type: multiTenant
  clusterLocatorMethods:
    - type: config
      clusters:
        - url: https://XXXXXXXXXXXXXXX.abc.us-east-1.eks.amazonaws.com
          name: backstage-tutorial-cluster # You can use any name you want
          authProvider: aws
          caData: ${EKS_CA_DATA}
```

If you're running a local cluster, the configuration is very similar, but it will use a `serviceAccount` auth provider instead of `aws`:

```yaml
kubernetes:
  serviceLocatorMethod:
    type: multiTenant
  clusterLocatorMethods:
    - type: config
      clusters:
        - url: ${MINIKUBE_ENDPOINT}
          name: backstage-tutorial-cluster # You can use any name you want
          authProvider: serviceAccount
          skipTLSVerify: true
          skipMetricsLookup: true
          serviceAccountToken: ${SECRET_TOKEN}
```

While there are several locator methods, you'll be using the `config` locator as it allows a secure connection to the clusters without hard-coding any credentials. You can read about the other methods [here](https://backstage.io/docs/features/kubernetes/configuration/#servicelocatormethod).

The `url` field indicates the endpoint where the app can communicate with the Kubernetes cluster. To get the endpoint from any Kubernetes cluster, use [kubectl](https://kubernetes.io/docs/reference/kubectl/) and run `kubectl cluster-info`. The output will look like this:

```shell
Kubernetes control plane is running at https://XXXXXXXXXXXXXXX.abc.us-east-1.eks.amazonaws.com
CoreDNS is running at https://XXXXXXXXXXXXXXX.abc.us-east-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

When connecting to AWS clusters, you need to provide CA data as part of the authentication. To get the CA data, you can either look up the details in the Amazon EKS cluster overview or use the AWS CLI. If you are using the CLI, run `aws eks describe cluster --name _your_cluster_name_`. You'll find the CA date in the response from `cluster.certificateAuthority.data`.

For a local cluster, just run the following to get the secret from the service account:

```shell
kubectl -n default get secret $(kubectl -n default get sa default -o=json \
| jq -r '.secrets[0].name') -o=json \
| jq -r '.data["token"]' \
| base64 --decode
```

To recap, for AWS, the whole configuration in `app-config.yaml` looks like this:

```yaml
kubernetes:
  serviceLocatorMethod:
    type: multiTenant
  clusterLocatorMethods:
    - type: config
      clusters:
        - url: https://XXXXXXXXXXXXXXX.abc.us-east-1.eks.amazonaws.com
          name: backstage-tutorial-cluster # You can use any name you want
          authProvider: aws
          caData: ${EKS_CA_DATA}

aws:
  mainAccount:
    roleName: backstage-tutorial-role  
    region: us-east-1
  accounts: # Add any additional accounts here if needed
  accountDefaults: # Add defaults here if needed
```

#### Configuring the Component

You've configured the Backstage application to communicate with the Kubernetes cluster and identify pods associated with specific components. The Backstage app also needs components that reference running pods for monitoring purposes. You can establish this link between components and pods through annotations. If a component lacks a Kubernetes annotation, the web app will prompt you to add one when you access the component (as shown in the first image after configuring the plugin). This annotation essentially specifies which pods within the cluster are relevant to the given component.

The following is an example of a component with the minimum required data:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: example1
  description: something
  annotations:
    backstage.io/kubernetes-id: backstage-app
    backstage.io/kubernetes-namespace: backstage
spec:
  type: service
  lifecycle: experimental
  owner: user:default/guest
```

Annotations beginning with `backstage.io/` are reserved for core components. The `kubernetes-namespace` annotation focuses the search for the pod on the appropriate namespace. The `kubernetes-id` annotation is a unique identifier for the component and is also used to match the running pod using label selectors. If this annotation does not exactly match the label in the pod, it won't be associated with your component. This detail is easy to overlook and could result in hours of troubleshooting.

For example, if you are deploying your pod with Terraform, your code needs to annotate the Kubernetes resources like this:

```hcl
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


resource "kubernetes_deployment" "backstage" {  
  metadata {  
    name      = "backstage"  
    namespace = "backstage"  
    labels = {  
      app = "backstage"
      "backstage.io/kubernetes-id" = "backstage-app" 
    }  
  }  
  
  spec {  
    replicas = 1  
  
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
  
      spec {  
        container {  
          image = "${var.BACKSTAGE_ECR}:${var.COMMIT_SHA}"
          image_pull_policy = "Always"
          name  = "backstage"  
          port {  
            container_port = var.APP_PORT  
          }  
          env_from {  
            secret_ref {  
              name = kubernetes_secret.backstage_secrets.metadata[0].name  
            }  
          }  
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
            value = aws_rds_cluster.aurora_postgres.port  
          }  
          env {  
            name  = "POSTGRES_DB"  
            value = "backstage"  
          }
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

This is just an example. The important part here is the use of labels to link resources to the Backstage component that will be associated with them. Your configuration might look different, and if you are not using infrastructure as code (like in minikube), you'll need to [deploy the pod manually](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/). The most important thing to remember here is that the `backstage.io/kubernetes-id` label has to match that of the catalog.

Now that the pod has the appropriate annotations, the easiest way to register a component that points to that pod is to create a new entry in the catalog section of `app-config.yaml` as follows:

```yaml
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]
  locations:

    # Local example template
    - type: file
      target: ../../examples/components/example1.yaml
      rules:
        - allow: [Component]
```

This config block points to a YAML file called `example1.yaml` in the `examples/components` folder in your Backstage app project folder. You'll need to create a file with that name. When doing so, you can use the YAML component example provided right at the beginning of the "Configuring the Component" section.

There are other ways to register components. If they are stored in a repository, you can [register them manually](https://backstage.io/docs/features/software-catalog/#manually-register-components) from the Backstage app. You can also create [custom software templates](https://backstage.io/docs/features/software-templates/writing-templates) for components that already have the appropriate documentation and register these to the catalog during the creation process.

To summarize, you first installed the Kubernetes plugin on both the frontend and backend. You then configured the app to be able to locate and connect to the Kubernetes cluster. You registered a component that points to the pod you want to monitor and used the label selector of the catalog entry to make the pod identifiable.

While this is an involved process with many steps, you only have to do most of them once, and you can automate the catalog registry using software templates. By doing this with every component that uses Kubernetes, you can easily navigate to their pages and see the health of the pods associated with them.

![Example of Kubernetes component working](https://i.imgur.com/nCf9TaS.png)

This kind of visibility is extremely valuable for administering services.

## Conclusion

You’ve now learned how to use your application to monitor specific Kubernetes pods while leveraging the power of the software catalog in Backstage. 

The examples here provide a tested foundation that should prevent you from getting lost if you transition from a proof of concept to production. While there's room for improvement and more creative solutions to the same problem, you now have a reliable starting point. Hopefully, this will save you several hours of digging around source code and stale GitHub pages, and you can use that time saved to keep improving your app and contributing back to the [open source project](https://github.com/backstage/backstage/blob/master/CONTRIBUTING.md).

Continue reading the next part of the series to learn how to deploy Backstage to a Kubernetes cluster.

---
title: "Evaluating Backstage 2: How Backstage Works"
authors:
  - Alexandre Couedelo
---
At its core, Backstage excels at bringing together an array of diverse tools, services, and essential information, all under one roof. This unified and customizable UI streamlines development workflows, improving productivity and empowering developers to thrive in their roles.

As an internal developer platform, Backstage fulfills the dream of a central hub with all the tools and information developers need. The days of platform engineers struggling to maintain their custom-made platforms may be over. But before throwing yourself single-mindedly into adopting Backstage, this article invites you to look at its inner workings and evaluate its offerings.

This is the second installment of the "Evaluating Backstage" series. The first article focused on [how and why Backstage was developed](https://www.kosli.com/blog/evaluating-backstage-why-backstage/). This article focuses on how Backstage works. You'll learn about Backstage's essential concepts and core offerings, and you'll also delve into Backstage's plugin architecture. Finally, as no evaluation is complete without addressing security concerns, the last section explores Backstage's security model.

## Backstage's Essential Concepts

Backstage is a software catalog, or a central repository that lists and categorizes software components your company develops or uses. You could compare Backstage to a phone book listing all microservices, desktop apps, mobile apps, and web applications, but it's actually much more than that. Backstage also serves as a frontend for all your developer tools, thanks to a modular plugin ecosystem that gathers your tools in a central place within the catalog so you can easily find them.

![Backstage software catalog](https://i.imgur.com/Q9yhKPH.png)

Backstage is highly customizable, so companies can adapt it to their exact needs. The code is open source, and when [installing Backstage](https://backstage.io/docs/getting-started/), you generate a template application that you can customize and extend with plugins. Backstage is not a black-box application where you install plugins; rather, you maintain your own source code and can modify it as needed.

Backstage is also a developer-centric platform that uses contemporary configuration management best practices, namely [GitOps](https://www.redhat.com/en/topics/devops/what-is-gitops#:~:text=GitOps%20uses%20Git%20repositories%20as,set%20for%20the%20application%20framework.). The catalog's definition is stored in Git repositories in YAML format (using a Kubernetes-style manifest). Owners of each Git repository are expected to write catalog configurations for the software they are building or maintaining, following the [Backstage system model](https://backstage.io/docs/features/software-catalog/system-model/).

### Backstage System Model

The system model is a conceptual representation of the data inside the Backstage software catalog. It defines all the high-level concepts (system, component, resource, API, and so on) that constitute the catalog and the possible relationships between its constituents. The following diagram illustrates the system model:

![Backstage system model by Spotify](https://i.imgur.com/rjV5wUX.png)
*[Image courtesy of Backstage](https://backstage.io/docs/features/software-catalog/system-model/)*

In a way, the system model is similar to a class diagram or a database schema. Understanding the model will help you configure the software catalog.

### Backstage Software Catalog

A software catalog allows you to manage your software assets (such as your frontend and backend applications as well as databases) in a structured and consistent manner, providing transparency and accountability throughout your company's ecosystem. Backstage allows you to easily locate all systems, resources, components, and features and see how they relate. Understanding these relationships helps you navigate the system and identify dependencies between applications. Ultimately, the services catalog can clearly show an overview of how a company's ecosystem works. It's important to keep track of who is responsible for each item in the catalog, so Backstage lets you assign ownership to a department, team, or even a person, and add ways to contact them.

![Backstage discoverability and accountability](https://i.imgur.com/78ZnreI.png)

When troubleshooting an issue, Backstage allows you to find the owner of a service and contact them quickly. On the other hand, as a developer, you can find out what systems your colleagues are working on and learn more about those systems.

The central component defined in the catalog is a *system*. A system is a set of elements that work together to provide some features. The following snippet is an example of how a system can be defined in the catalog:

```yaml
# Defining a system
apiVersion: backstage.io/v1alpha1 # Kubernetes-style manifests
kind: System
metadata: # Provide context
  name: audio-playback
  description: Audio playback system
spec: # Define relations
  owner: team-c # Define accountability
  domain: playback
```

Once you define the system, you must complete the catalog with more information about that system. A system can be part of a group of systems called domains. A system has components (pieces of software) and resources (services it needs to run). Both components and resources represent the structure of a system. A system also provides or consumes various APIs that define what it offers and its dependencies on other systems.

You can define all entities described in the diagrams in the same way. The following example defines a component for the `audio-playback` system above. In this example, let's imagine that this component provides an API (`playback-order-api`) and consumes a database (`playback-db`):

```yaml
# Defining a component
apiVersion: backstage.io/v1alpha1 # Kubernetes-style manifests
kind: Component
metadata: # Provide context
  name: playback-order
  description: Playback Order
  tags:
    - java
    - playback
spec: # Define relations
  type: service
  lifecycle: production
  owner: team-c # Define ownership
  system: audio-playback # Define the service relationship
  dependsOn:
    - resource:default/playback-db # This component uses a DB
  providesApis:
    - playback-order-api # The component is providing an API
```

The next step is to define the resource (`playback-db`) and API (`playback-order-ap`). If you're interested, you can find those configurations in detail [on GitHub](https://github.com/xNok/backstage-demo/blob/main/backstage/examples/audio-playback.yaml). But the most important thing is that you can see the model in Backstage once those entities are ingested in the catalog. The following is the Catalog Graph page, where you can see the described model system:

![Backstage Catalog Graph page](https://i.imgur.com/eO8CenM.png)

Once you define all the entities (system, components, resources, APIs, and so on), you get a broad overview of your company's ecosystem.

## What Does Backstage Offer?

Backstage comes with a few really interesting features right out of the box.

### Technical Documentation Hub

Backstage enhances the notion of a service catalog by offering the option to host documentation as code. Specifically, developers are encouraged to write their documentation using Markdown inside their code repository. Backstage will then render and host those pages in the service catalog:

![Backstage TechDocs](https://i.imgur.com/xkqKpEx.png)

Backstage's technical documentation feature (TechDocs for short) is built on top of [MkDocs](https://www.mkdocs.org/), a popular documentation as code static website generator. When writing documentation, you can leverage many preexisting [MkDocs plugins](https://github.com/mkdocs/catalog) and beautify your documentation with additional styling (not supported by traditional Markdown) and diagrams.

### Resource Search Engine

When you combine documentation with a service catalog, you have all the information you need in one place. To help sort through it, Backstage includes a powerful search engine that allows developers to easily find the resources and information they need. This search engine can search the service catalog, documentation hub, and other resources such as Confluence or Stack Exchange via plugins. Backstage's search engine can index multiple sources of information, so it can provide exhaustive results from those sources. Having a single search engine in Backstage makes it easier to find information quickly, reduces the time wasted navigating between different sources of information, and helps developers work more efficiently and effectively.

![Backstage search engine](https://i.imgur.com/9IFYMZK.png)

### Software Templates and Golden Paths

Backstage provides a self-service [software template](https://backstage.io/docs/features/software-templates/) solution for a developer to start a project with the necessary tools and resources. For instance, a template could allow a developer to quickly create a Git repository with a skeleton application in a chosen programming language.

Software templates aim to provide the key set of steps that developers should follow when starting new projects, and most steps can be automated. Those key steps are referred to as [Golden Paths](https://engineering.atspotify.com/2020/08/how-we-use-golden-paths-to-solve-fragmentation-in-our-software-ecosystem/) and aim to ensure developer autonomy (no gatekeeping to start a project) while ensuring standards and best practices are followed (a template is provided).

![Backstage software template](https://i.imgur.com/WbftMh4.png)

You can also create a software template to automate infrastructure tasks and provide on-demand databases, event queues, Kubernetes clusters, and so on. The software template feature offers a UI for developers to say, "I need something." Behind the curtain, the team maintaining the template (most likely the infrastructure team) can implement some mechanism to process that request automatically. For instance, choosing a software template can result in a pull request to a repository with a Terraform configuration, Kubernetes manifest, or Crossplane manifest.

The software template feature can turn Backstage into an internal developer platform (IDP) that provides self-service capabilities for developers to independently obtain anything they require to create and run their applications.

## Adopting Backstage: Understanding Plugins

Backstage is a web application with a [Node.js](https://nodejs.org/en) backend and a [React](https://react.dev/) frontend. A basic understanding of those two ecosystems, including TypeScript, is required to be able to customize the application and integrate plugins.

All plugins are Node modules that can be installed on the Backstage application. When you first start your Backstage journey, you generate a template app that's built using the core plugins mentioned in the previous section.

The frontend part of Backstage is built using React components provided by plugins. Similarly, the backend registers API endpoints provided by backend plugins. So, any third-party integration needs two or more plugins, typically at least a frontend one and a backend one. The endpoints provided by the backend plugin are used by the frontend plugin to fetch the information to display.

![Backstage composable architecture](https://i.imgur.com/Es4W5Sc.png)

### Backstage Community Plugins

You can find the list of plugins on the [Plugin Marketplace](https://backstage.io/plugins/), and you should browse the list to discover more about what features Backstage can offer. You may notice that the four features discussed above (software catalog, technical documentation, search engine, and software templates) are plugins. Indeed, in Backstage, everything is a plugin.

![Backstage Plugin Marketplace](https://i.imgur.com/59WHIOR.png)

The following are a few examples of notable plugins:

* You can extend the search engine with sources such as [Stack Exchange](https://github.com/backstage/backstage/tree/master/plugins/stack-overflow) or [Confluence](https://github.com/K-Phoen/backstage-plugin-confluence)
* You can integrate CI/CD to see the state of your pipeline using plugins from [Jenkins](https://github.com/backstage/backstage/tree/master/plugins/jenkins), [CircleCI](https://github.com/backstage/backstage/tree/master/plugins/circleci), or [GitLab](https://github.com/immobiliare/backstage-plugin-gitlab)
* You can integrate deployment tools and platforms such as [Kubernetes](https://backstage.io/docs/features/kubernetes/) and/or [Argo CD](https://roadie.io/backstage/plugins/argo-cd/)

The list of plugins keeps growing. Spotify, which develops Backstage, has a dedicated team working on plugins. The project maintains open source plugins and recently announced a new set of [premium plugins](https://backstage.spotify.com/plugins/) under development.

In many cases, you should consider creating your own plugins to extend Backstage and integrate seamlessly with your ecosystem. Generally, with custom plugins, you can add new functionality, modify existing features, customize the UI, integrate with third-party systems, and set up custom workflow steps with the software templates plugin.

### Backstage Custom Plugins

The goal here is not to teach you how to create your own plugins, but to provide an overview of how plugins work so you can evaluate the effort it takes to create your own.

There are two main reasons you would create your own plugins: being able to customize the UI—especially dashboards and pages of the catalog—and being able to integrate with external systems via APIs. In the first case, all you need to do is create a frontend plugin. As mentioned, third-party integrations require both a backend plugin and a frontend plugin since you need to call external APIs and display that information in Backstage.

On the frontend side, writing a custom plugin is equivalent to writing a [React](https://react.dev/) component library. A page is an aggregation of React components. When opening a catalog page, you can see four cards, each of which is a React component: About, Relations, Links, and Has subcomponents. Each tab (CI/CD, APIs, Dependencies, and so on) will display more components from various plugins to compose a page.

For instance, you can see this on the `playback-order` page below:

![Backstage Catalog Page decrypted](https://i.imgur.com/CeilU05.png)

Even if you are not a React expert, understanding how this page is defined should be straightforward. The Overview tab definition you saw above looks like this:

```js
const overviewContent = (
  <Grid container spacing={3} alignItems="stretch">
    {entityWarningContent}

    <Grid item md={6}>
      <EntityAboutCard variant="gridItem" />
    </Grid>

    <Grid item md={6} xs={12}>
      <EntityCatalogGraphCard variant="gridItem" height={400} />
    </Grid>

    <Grid item md={4} xs={12}>
      <EntityLinksCard />
    </Grid>

    <Grid item md={8} xs={12}>
      <EntityHasSubcomponentsCard variant="gridItem" />
    </Grid>
  </Grid>
);
```

In the code above, a `Grid` system is used to organize, display, and define the page structure and the four components evoked earlier (`EntityAboutCard`, `EntityCatalogGraphCard`, `EntityLinksCard`, and `EntityHasSubcomponentsCard`).

Composing new tabs and pages using the frontend components of plugins is a task for the platform engineering team or developers maintaining Backstage. As you can see, you have to write some code (piece together code would be more accurate), but in return you get a lot of control over what your instance of Backstage looks like, meaning your company's experience with Backstage will be tailored specifically to your needs.

On the backend side, plugins are mostly used to fetch information to create third-party integrations. For example, with the [Kubernetes plugin](https://backstage.io/docs/features/kubernetes/), which is listed as a core plugin, the UI needs to access information about the cluster:

![Backstage Kubernetes plugin](https://i.imgur.com/Y0WvlCg.png)

The Kubernetes React component first needs to discover the URI of its backend counterpart. When initializing, the plugin uses the discovery API that provides the endpoint for the requested backend (steps 1, 2, and 3 in the diagram below). Once the frontend knows about the backend URI, it can make a call to fetch the data. The request is sent to the application and handled by the router that dispatched the request to the right backend component (steps 4 and 5). Finally, the backend handles communication with the third-party API and returns the requested data. In the case of the Kubernetes plugin, this means the dedicated backend plugins are responsible for communicating with the Kubernetes API.

![How third-party integration works in Backstage](https://i.imgur.com/QNUF6Uq.png)

Backend plugins provide a reliable and efficient mechanism for frontend components to interact with backend services, enabling seamless integration with third-party APIs and retrieving necessary data. The backend plugins can also integrate some authorization mechanisms to control users' access.

This concludes the tour of how to customize Backstage applications and integrate plugins to create a tailored experience. The frontend uses React components from plugins, while the backend registers API endpoints for fetching and displaying information. With its composable architecture, Backstage offers customizable dashboards and third-party integrations, providing control over the platform's appearance and facilitating data collection and aggregation in the UI.

## Adopting Backstage: Security Concerns

Security for an internal developer platform is very important, as the platform is a central hub or collection of much of the company's internal information. As Backstage is based on plugins pulling and processing information from various other systems, it can also be subject to security risks.

Generally, a threat model helps identify and mitigate potential security threats. It protects sensitive information, ensures secure access, addresses infrastructure security, fosters secure development practices, and helps you meet [compliance requirements](https://www.kosli.com/audit-compliance/). Backstage's [threat model](https://backstage.io/docs/overview/threat-model/) includes a trust model, which identifies the following three actors:

* An **integrator** deploys, operates, and configures Backstage and is fully trusted.
* **Internal users** (authenticated users who are part of the company) are partially trusted. This means that although they have access to company resources, they cannot be fully trusted to maintain the confidentiality and integrity of the data. Internal user access should be restricted to only the data needed, and their inputs/configurations should be validated.
* **External users** (anyone else not part of the company) are not trusted and should not have any access to Backstage.

![Backstage trust model](https://i.imgur.com/W4FtCwI.png)

### Security Recommendations for Authentication

Internal users must be identified before being granted access to Backstage. The authentication process for Backstage relies on using the [auth plugin](https://backstage.io/docs/auth/) that supports various OAuth 2.0 provider integrations like GitHub, GitLab, Azure, and Google. The Backstage auth plugin doesn't protect against unauthorized access from external users and only focuses on internal users.

So, the main threat comes from external users. It's recommended to deploy Backstage behind identity-aware proxies such as GCP IAP, Cloudflare Access, or AWS's ALB to prevent all access from external users to Backstage. The Backstage API should be secured using Backstage's JWT tokens that assert a user's identity, and all backend-to-backend communication should also use JWT tokens with a pre-shared signing secret, especially when exchanging information with third-party APIs.

### Security Recommendations for Core Plugins

Backstage offers authentication and authorization and emphasizes that it's the integrator's responsibility to manage that configuration when deploying a Backstage instance. The model assumes that Backstage is deployed securely, preventing external access, and that internal users can be trusted and have access by default to all resources in the catalog.

The integrator needs to restrict permissions if needed and validate that the recommendation of the threat model is enforced for the core plugins. The threat model and integrator are particularly important for implementing security measures in the following areas of Backstage:

* **Software catalog:** Registering user and group entities should not be allowed to be delegated to identity providers via provisioners. Ideally, you should apply rules to limit the allowed entity kinds and validate configurations, protecting against the creation of many resources or compromising data integrity.
* **Software templates:** You should be especially cautious when using the [Scaffolder plugin](https://backstage.spotify.com/learn/onboarding-software-to-backstage/setting-up-software-templates/10-scaffolder-actions/), as it may have elevated permissions to create resources on behalf of users. Backstage also suggests auditing and verifying installed scaffolding actions (automated steps for templates).
* **TechDocs:** The TechDocs frontend applies strict sanitization to generated HTML content, so the risk is more on the backend side if you use external storage such as Google Cloud Storage, Amazon S3, or Azure Blob Storage (the recommended setup for companies with a lot of documentation). In that case, the integrator is responsible for securing access to the storage.
* **Proxy:** Backstage offers the option to configure proxies so the frontend can access data from third parties. This is a simple alternative to creating backend plugins for Backstage. However, it comes with the risk of exposing too much of the third-party API and not providing identity-based access control. To prevent that issue, you should create backend plugins to ensure the principle of least privilege is applied and offer better authorization workflows with third parties.

Community plugins should be assessed carefully and not blindly trusted, as no default restrictions are in place to restrain what a plugin can access from the catalog.

## Conclusion

You should now have a good grasp of how Backstage works and how its pieces fit together. Backstage’s system model is the core representation of the catalog configuration and serves as the platform's backbone. It dictates how information is presented, accessed, and managed by users. While simple, every user should understand this model to ensure the catalog is appropriately structured.

Backstage goes beyond a service catalog by integrating technical documentation and a search engine, offering developers a self-service solution with software templates. The platform also provides modularity through plugins, though they require some programming knowledge to implement. However, you don't need to be a React expert to assemble a UI component, and you have complete control over what to display where. You also learned about some security concerns you should consider before going live with the platform.

If you’ve followed along since part one, you’ve seen the "why" and the "how" behind Backstage. Maybe you’re already sold on the platform and want to start implementing it immediately. However, you can’t fully evaluate a tool until you see how it compares to its competitors. If you’re curious to see how it stacks up, you can continue your journey with the third installment, [Evaluating Backstage: Backstage vs. Competitors](https://www.kosli.com/blog/evaluating-backstage-backstage-vs-competitors/).
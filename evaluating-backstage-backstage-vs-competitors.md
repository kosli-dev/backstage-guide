---
title: "Evaluating Backstage: Backstage vs. Competitors"
authors:
  - Artem Oppermann
---
Developer portals are no longer a novelty but a necessity for organizations that offer software services. The portals centralize and streamline the developer experience with essential tools, including API documentation, SDKs, sample code, and technical support. Backstage is one example of a developer portal, but it's not the only option. Before deciding if you'd like to adopt Backstage, it's crucial that you consider how it compares to its competitors. 

This is the third part of the "Evaluating Backstage" series. The [first article covered the "why,"](https://www.kosli.com/blog/evaluating-backstage-why-backstage/) and the [second article focused on the "how."](https://www.kosli.com/blog/evaluating-backstage-how-backstage-works/) This third installment explores how Backstage compares to some other platforms for building developer portals:

* [OpsLevel](https://www.opslevel.com/)
* [Port](https://www.getport.io/)
* [Cortex](https://www.cortex.io/)

Each of the platforms is assessed based on its core features, ease of use in terms of user interface (UI) and user experience (UX), learning curve and complexity, integration capabilities with existing infrastructure, customizability and extensibility, collaboration and communication features, quality of documentation and support, and resource requirements. These criteria reflect the daily practicalities of these platforms. 

## Backstage

![Backstage](https://i.imgur.com/QKhnxYv.png)

[Backstage](https://backstage.io/) is an open source platform that facilitates the creation of developer portals. The platform acts as a centralized catalog for all software assets. It's designed to bring structure to the often chaotic world of microservices and infrastructure, which should ultimately help development teams maintain code quality and enable them to ship code more quickly.

### Core Features

Backstage works particularly well in large companies with substantial developer tooling. Its core feature, the [Backstage software catalog](https://backstage.io/docs/features/software-catalog/), can effectively manage a wide array of software, including but not limited to microservices, websites, and machine learning models. This tool provides a uniform overview of the entire software landscape of an organization, along with readily available metadata. This makes it easier to discover and manage software components—a significant advantage for large-scale operations.

Backstage's [software templates](https://backstage.io/docs/features/software-templates/) allow developers in large companies to create software components in a consistent and standardized manner. This enhances overall efficiency and cohesion in expansive development environments.

Backstage also features [TechDocs](https://backstage.io/docs/features/techdocs/), a tool that streamlines the creation, maintenance, and usage of technical documentation—an often complex task for large organizations. TechDocs adopts a "docs like code" approach that allows documentation to be created using the same principles as coding. This makes the documentation process more intuitive and straightforward. This further enhances the benefits for large companies with extensive developer tooling.

### UX/UI

In terms of UX/UI, Backstage follows an iterative design approach, which means the platform is continually evolving and improving. The design team behind Backstage is also very collaborative and transparent. The team works closely with the community to make sure that the platform meets the needs of its users. On the other hand, however, some users have reported that the UI and UX of Backstage could be more visually appealing and still need some maintenance. For instance, inconsistencies in the platform's navigation across different sections are often highlighted. Some users have found it difficult to locate certain features due to the inconsistent placement of navigation elements, which interrupts the user flow and makes the experience less intuitive.

### Learning Curve and Complexity

Backstage is engineered with a focus on user-friendliness and ease of use. It has a variety of features, such as a software catalog, software templates, and technical documentation, all strategically designed to facilitate developers in swiftly starting and operating their projects. Despite the platform's robust feature set, there’s lots of help when it comes to getting started with Backstage, thanks to comprehensive documentation, intuitive tutorials, and informative blog posts. These resources help reduce the learning curve, ensuring developers can leverage the platform's capabilities right from the get-go.

### Ease of Integration with Existing Infrastructure

Backstage is quite versatile in terms of ease of integration with existing infrastructure. Not only does Backstage provide tooling to build Docker images, facilitating deployment on a wide range of infrastructures, but it can also be installed as a Node.js package. This dual capability further broadens the range of hosting services it can be deployed to. The platform is designed to be deployed in the same way as other software at your organization, which makes it easy to integrate with your existing infrastructure.

### Customizability and Extensibility

Backstage's customizability is one of its strongest advantages. The platform has a growing [ecosystem of open source plugins](https://backstage.io/plugins/) that expand its customizability and functionality. These plugins allow platform engineers to easily integrate new tools and services, such as Jenkins for continuous integration, Grafana for data visualization, or Kubernetes for container orchestration. It also provides the capability to extend the functionality of existing plugins, creating a more robust, custom, and efficient infrastructure.

### Collaboration and Communication

Backstage also has the capacity to foster improved communication and collaboration among developers. Its software catalog feature allows developers to understand the software landscape within the organization. This can reduce duplicate efforts and enhance team cooperation. Its software templates promote the use of best practices, streamlining project initiation and enforcing consistency across the organization.

Backstage's plugins can create a unified platform for developers to interact with a myriad of tools and services, which directly improves collaboration and communication. While other tools might offer certain standalone features, Backstage excels at seamlessly integrating these functionalities.

### Documentation and Support

The platform offers a variety of resources to assist users and developers. These include a [Discord server](https://discord.com/servers/backstage-687207715902193673) for support and project discussions, a [Stack Overflow section](https://stackoverflow.com/questions/tagged/backstage) for browsing or asking questions, and a list of ["good first issues"](https://github.com/backstage/backstage/contribute) for those who wish to contribute to the project. An [FAQ](https://backstage.io/docs/FAQ/) is available for quick reference. Updates and announcements are shared via a [blog](https://backstage.io/blog/) and an [email newsletter](https://info.backstage.spotify.com/newsletter_subscribe). The [Backstage community hub](https://backstage.io/community/) provides additional resources, including community sessions and recordings.

#### Resource Requirements

Backstage has several prerequisites for getting started. These include access to a Unix-based operating system such as Linux, macOS, or Windows Subsystem for Linux; an account with elevated rights to install dependencies; and having [curl](https://curl.se/) or [Wget](https://www.gnu.org/software/wget/) installed. Once you have these in place, Backstage can be deployed using [npm](https://www.npmjs.com/) packages and run with a SQLite in-memory database and demo content. However, it's important to note that running Backstage in a real environment typically involves containerizing the components. This may require additional effort and resources to set up. All of this means that getting started with Backstage can be a little complicated.

## OpsLevel

![OpsLevel](https://i.imgur.com/gpMQXeQ.png)

[OpsLevel](https://www.opslevel.com/) is a developer portal designed to enhance the efficiency and standards of high-performing engineering teams. The portal provides a centralized location for cataloging all microservices, systems, teams, and tools, which offers visibility into the entire architecture at a glance.

### Core Features

One of the core features of OpsLevel is the ability to build a complete software catalog in minutes just by connecting your Git forge. Teams can quickly gain an understanding of the services that exist in their ecosystem. Another core feature of OpsLevel is its ability to track every software object running in production and the team responsible for it. This provides 
visibility into the software architecture and can help teams understand its current health and maturity at a glance. OpsLevel also centralizes technical and API documentation, which makes it easier for developers to breeze through feature development and code fixes.

### UX/UI

OpsLevel's UI and UX are designed with clarity and functionality in mind. The UI is sleek and intuitive, which makes navigation pretty simple, even for new users. The arrangement of data and controls contributes to a positive UX. This allows users to find and manage microservices efficiently. On the flip side, the richness of information and controls that OpsLevel provides can potentially lead to information overload, particularly for those who are new to the system.

### Learning Curve and Complexity

OpsLevel offers a suite of features that facilitate microservice management. It includes a centralized [catalog](https://www.opslevel.com/microservice-catalog) that provides a database of all the microservices in your organization, along with vital information like ownership, technology stack, and health metrics. However, this breadth of functionality can result in a steep learning curve. Users may find OpsLevel's extensive feature set overwhelming initially, especially those unfamiliar with microservice architectures. The platform's reliance on multiple external service integrations for optimal functionality can further add to its complexity. Such integrations include commonly used CI/CD tools like [Jenkins](https://plugins.jenkins.io/opslevel/) and [GitHub Actions](https://docs.opslevel.com/docs/github-actions-integration), alerting and incident management systems like [PagerDuty](https://docs.opslevel.com/docs/pagerduty), and version control platforms like [GitHub](https://docs.opslevel.com/docs/github-integration). Despite its potential advantages, teams must consider their preparedness to navigate this learning curve and embrace its complexity before deciding to adopt OpsLevel.

### Ease of Integration with Existing Infrastructure

OpsLevel can also integrate with existing infrastructure, such as Kubernetes, and popular Git providers like GitHub, GitLab, Azure DevOps Git, and Bitbucket. However, setting up these integrations may require some effort. For example, to [set up the GitHub integration](https://docs.opslevel.com/docs/github-integration), users must go to the Integrations tab in OpsLevel, hit the "New Integration" button, click the GitHub integration card, choose the GitHub App option, and follow the instructions within GitHub to install the OpsLevel GitHub App.

### Customizability and Extensibility

OpsLevel features can be tailored to match specific workflows, and the ability of the platform to integrate with various external services bolsters its extensibility. All in all, this makes OpsLevel a versatile tool for microservice management, but this customization does come with challenges. Notably, the requirement to tailor the tool to the specific needs of the company can demand significant initial setup time. Moreover, its extensibility, while providing enhanced functionality, depends on integration with other services. For development teams that are not currently using these services or those who resist adopting them, this can be a barrier to fully utilizing OpsLevel's capabilities. As a result, development teams should evaluate their resources and readiness for such customization and integration efforts before implementing OpsLevel.

### Collaboration and Communication

OpsLevel significantly enhances collaboration and communication within DevOps teams. It serves as a unified platform for tracking and managing microservices, which provides increased visibility across the team and fosters a shared understanding of the system's state. By centralizing information and promoting clear communication, OpsLevel supports cohesive team operations. This ultimately facilitates a more efficient and responsive DevOps workflow.

### Documentation and Support

OpsLevel centralizes technical and API documentation that developers need to breeze through feature development and code fixes. OpsLevel also provides a [Getting Started](https://docs.opslevel.com/docs/getting-started) section on its website that includes explanations for various features and integrations, like setting up single sign-on, setting up checks for services, and interacting with the GraphQL API.

### Resource Requirements

OpsLevel is a web-based software-as-a-service (SaaS) platform that runs on the cloud and is accessible via a web browser. The primary resources you need to utilize OpsLevel are a stable internet connection and a current web browser. There are typically no specific system requirements beyond what's needed to efficiently run a modern web browser. To leverage OpsLevel to its fullest potential, you would need a software infrastructure that OpsLevel supports, like microservices or APIs, and the capacity to integrate OpsLevel with your existing systems, such as version control systems, CI/CD pipelines, and alerting and monitoring tools.

## Port

![Port](https://i.imgur.com/cIGMNeV.png)

[Port](https://www.getport.io/) is a developer portal designed to streamline and enhance the software development process. It provides a context-rich software catalog, maturity and quality scorecards, and comprehensive developer self-service actions. Port is designed to automate DevOps routines, reduce clutter around infrastructure, and save time spent on understanding, communicating, and managing DevOps assets.

### Core Features

The key features of Port include a software catalog, live data inclusion, Kubernetes visibility, and self-service developer actions. The software catalog is populated via an API and allows you to create your own data model with unlimited blueprints and dependencies between any data model elements. It includes rich properties, facilitating data modeling with no coding involved.

Additionally, Port includes a Kubernetes catalog, providing visibility for Kubernetes objects and support for multiple clusters and custom resource definitions (CRDs) like [Argo CD](https://argo-cd.readthedocs.io/en/stable/) and [Istio](https://istio.io/). Port also empowers workflow automation with event-based subscriptions, TTL cron jobs, scorecards, and live integration with tools such as Datadog, Jira, and PagerDuty. It supports native integration with CI/CD data.

[Self-service experiences](https://docs.getport.io/create-self-service-experiences/) are designed to drive developer productivity by allowing developers to perform actions like scaffolding a service or provisioning a cloud resource independently. These actions bring consistency and repeatability. This ensures that developers follow best practices intuitively and clearly.

However, it's important to note that Port lacks a built-in, docs-like-code solution (like TechDocs in Backstage) to enable developers to write their documentation in Markdown files that live together with their code—and get a nice-looking doc site in Port. This means that teams who prioritize integrated, easily accessible documentation may find Backstage more suitable for their needs.

### UX/UI

Port offers an aesthetically pleasing user interface with a modern design and layout. On the flip side, it can potentially be overwhelming for some users. The platform is packed with features and tools, which can lead to a complex and dense user experience. The multitude of options and settings, combined with the extensive cataloging capabilities, might be especially confusing for new users or those not familiar with such comprehensive platforms.

### Learning Curve and Complexity

While Port offers a comprehensive suite of features and a high degree of customizability, it has been reported that the platform can be quite challenging to learn, especially for beginners. The complexity of the system, combined with the multitude of options and settings, can make the initial learning curve steep. Users might find themselves daunted by the array of integrations, the process of setting up custom workflows, and the task of tailoring the platform to their specific needs.

### Ease of Integration with Existing Infrastructure

Integrating third-party tools into Port can be a complex process, as you can see from the steps required for [AWS integration](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/aws/Installation/). Users must first ensure they have their Port credentials and necessary software installed, such as the AWS CLI and AWS SAM CLI. The integration process involves running scripts in the terminal, cloning the Terraform template, initializing Terraform requirements, and deploying the AWS exporter. Users must also set up EventBridge rules, invoke the AWS exporter Lambda function, and modify various parameters. This process requires a deep understanding of both Port and AWS systems, making it a potentially challenging task for those unfamiliar with these platforms.

### Customizability and Extensibility

Port provides out-of-the-box integrations with a wide range of tools and also supports the creation of custom integrations, making it highly adaptable to various workflows and systems. This includes integrations with Git, CI/CD systems, Kubernetes, cloud platforms like AWS and GCP, and much more.

### Collaboration and Communication

Port promotes collaboration and communication through various strategies. It encourages active information sharing via presentations and internal conferences, fostering a positive and productive team environment. Its developer portal consolidates processes into a self-service model, reducing the need for constant communication. However, Port does not offer out-of-the-box solutions for knowledge exchange and surveys regarding developer experience. Knowledge exchange is an important component of collaboration, as it allows team members to share insights, learn from one another, and avoid duplicating efforts. Similarly, surveys are an effective way to gauge the developer experience, understand developers' needs, and make necessary improvements. The absence of these features in Port could potentially lead to gaps in understanding and communication within the team, which could impact productivity and overall team synergy.

### Documentation and Support

Port is committed to providing comprehensive and robust support to its users. This includes user-friendly documentation containing detailed guides, release notes, and blog articles. Beyond these resources, Port also maintains an active community where users can directly interact with engineers for support, provide input on product development, and connect with fellow users.

### Resource Requirements

Port is a hosted solution that simplifies the resource requirements for its users by eliminating the need for individual hardware infrastructure and complex software maintenance. However, users must still maintain a stable internet connection, operate compatible devices capable of running Port's web browser or application interface, and secure adequate user licenses based on the size of the team and the nature of the projects. Additionally, considerations around data storage, including backup and archival needs, as well as user training for proficiency in the platform, are important. These resource commitments are essential for fully leveraging the benefits of this SaaS solution.

## Cortex

![Cortex](https://i.imgur.com/5HlvuEF.png)

[Cortex](https://www.cortex.io/) is a developer portal designed to help engineering teams adopt and maintain microservices best practices.

### Core Features

Cortex offers several core features designed to streamline the software development process. It includes a developer portal that provides engineering teams with visibility into their services, thus enabling the delivery of high-quality software. The platform allows for integration with third-party Git and APM tools, facilitating the setup of a robust GitOps workflow to keep the system updated.

Additionally, Cortex provides an API that supports automation using Cortex as the definitive source for a [service catalog](https://www.cortex.io/products/service-catalog) and [scorecards](https://docs.cortex.io/docs/reference/basics/scorecards). This feature can help speed up the development of a resilient information architecture.

### UX/UI

Cortex's design includes an intuitive user interface that prioritizes ease of use and user experience. The layout of the platform is clean and organized, which makes navigation straightforward and allows users to quickly locate the tools and features they need.

### Learning Curve and Complexity

Cortex's documentation has a quick crash course on how to get started with Cortex, including how to connect to your third-party tools and set up a powerful GitOps workflow. Cortex also provides documentation on how to integrate with all of your favorite Git and APM tools, among others. This ensures that getting started with Cortex is not overly complicated for beginners.

### Ease of Integration with Existing Infrastructure

Cortex provides a broad range of integrations with popular data sources and third-party tools, including AWS, Bitbucket, and GitHub, to name a few. The platform aims to integrate smoothly with your existing infrastructure and tools. However, the complexity of integration can vary depending on the specific third-party tool involved. For example, integrating Cortex with AWS can be quite intricate due to the multiple steps required. This process demands a comprehensive understanding of both AWS and Cortex systems. It's important to note that while Cortex strives for seamless integrations, the level of difficulty can fluctuate based on the tool you're integrating with.

### Customizability and Extensibility

A significant disadvantage of Cortex is its limited customizability and extensibility due to the absence of plugin support. The lack of a plugin system restricts users from extending the functionality of the platform beyond its built-in features. This limitation can be a hindrance for teams that require specific functionalities or integrations that are not natively supported by Cortex. Without the ability to add plugins, users may find it challenging to tailor the platform to their unique needs or adapt it to evolving requirements. This lack of customizability and extensibility can limit the versatility of the platform and potentially impact its long-term suitability for some teams.

### Collaboration and Communication

Cortex is engineered to boost communication and collaboration within businesses. It serves as a hub for each team that fosters collective service ownership, speeds up developer onboarding, and improves cross-team collaboration. Beyond just facilitating communication, Cortex also emphasizes the celebration of team achievements and organizational progress by recognizing top-performing teams through awarding badges and notifying the team of wins. This can contribute to a positive work environment.

### Documentation and Support

Cortex provides documentation and support to streamline user onboarding and optimize the use of its features. This includes a quick start guide that helps users connect to third-party tools and establish a GitOps workflow. However, there can be instances where the documentation lacks detail. The lack of comprehensive instructions can make it challenging for users to fully understand how to properly integrate and use Cortex and third-party tools within their own systems. Such gaps can lead to potential misconfigurations, inefficiencies, and even security risks.

### Resource Requirements

Cortex can be deployed on-prem into your own Kubernetes cluster. To operate Cortex in a self-managed setup, specific resource requirements must be met. This includes two instances of the backend container, each needing 3.5 GB of memory and, ideally, two cores. Additionally, a single instance of the frontend container is required, which has a low memory demand of less than 500 MB since it functions as a static Nginx proxy. A crucial component is a Postgres database, version 9 or higher, equipped with 15 GB of storage and 4 GB of memory.

## Conclusion

As you can see, Backstage isn't your only option if you want to implement a developer platform. This article compared Backstage with three other notable developer tools: OpsLevel, Port, and Cortex.

Backstage stands out for its robust customizability and growing ecosystem of open source plugins, making it an attractive option for larger companies with extensive developer tooling. However, some users have expressed concerns about its UI/UX and perceived complexity. OpsLevel and Port offer solid integrations with existing infrastructure and considerable customizability. However, they have potential challenges, such as a steep learning curve and substantial setup and management requirements. On the other hand, Cortex is designed for ease of use and promoting collaboration but lacks plugin support, resulting in limited customizability and extensibility.

If you've followed along since the start of this series, you now know the why and how behind Backstage and seen how it compares to some other developer platforms. At this stage, you should know if Backstage is the right developer platform for you. If you're convinced, you can begin exploring its implementation in the first part of the next series, Implementing Backstage: Getting Started.

---
title: "Succeeding with Backstage 4 - Backstage as Part of a Broader Developer Productivity Engineering (DPE) Initiative"
authors:
  - Alexandre Couedelo
---
This final article in the "Succeeding with Backstage" series focuses on how you can incorporate Backstage as part of a broader developer productivity engineering (DPE) initiative. The previous parts dealt with [customizing the look and feel of Backstage](URL), [creating and maintaining custom plugins](URL), and [improving Backstage adoption](URL).

Developer productivity engineering (DPE) is a relatively new discipline that focuses on simplifying developers' tasks and promoting collaboration, aiming to enhance their overall experience. DPE initiatives prioritize keeping developers happy and productive, thus also preventing losses linked to unproductivity.

As you've seen throughout this series, Backstage is a platform created to improve the developer experience by providing a central place to catalog your entire software ecosystem and integrate with your stack to gather all the information in one place. So, it would seem natural that Backstage and DPE initiatives would complement each other. But do DPE and Backstage make a winning combination?

This article explores the concept of developer productivity engineering, how it fits into the software development landscape, and—most importantly—whether Backstage has the potential to be a game-changer for your DPE initiatives.

## What Is Developer Productivity Engineering?

DPE draws inspiration from lean manufacturing methods and is closely aligned with Agile and DevOps. It’s a relatively recent development in the evolution of software engineering best practices. DPE has emerged as a catalyst for shortening development task durations through analysis, task simplification, and standardization.

DPE as a practice was [initiated by Netflix in 2016](https://www.youtube.com/watch?v=PqgbJSgvqj0&ab_channel=Gradle) and then adopted by [Microsoft](https://www.youtube.com/watch?v=1xP72VhpT1U&ab_channel=Gradle), [Splunk](https://www.splunk.com/en_us/blog/learn/dpe-developer-productivity-engineering.html), and [Gradle](https://gradle.com/blog/developer-productivity-engineering-2020). DPE addresses the technology challenges that arise in fast growing companies. As the amount of code, the number of code repositories and dependencies, and the complexity of the tech stack increase, developers tend to spend less time in a productive and creative state. Instead, they become increasingly frustrated and delayed by unproductive tasks like waiting for the build process, responding to failing pipelines, debugging unclear errors, and resolving dependency issues.

The scope of DPE encompasses activities ranging from coding and building to continuous integration (CI) and testing. This is what sets it apart from other trends, such as DevOps, DevExp, and platform engineering. While these other approaches aim to provide a more comprehensive solution for delivering fast and reliable software, DPE focuses on the specific needs and challenges in the early phase of software development. It recognizes that developers should primarily invest their time in designing and writing software.

![What is DPE, and where should developers spend their time?](https://i.imgur.com/6rXGbGu.png)

### The Key Goals of Developer Productivity Engineering

In short, DPE aims to improve collaboration and delivery speed in software development. The key to a successful DPE initiative is identifying significant opportunities with a good return on investment (ROI) and ensuring that the adoption and cost of tools result in a proportional improvement in developers' experience and productivity.

DPE advocates for a data-driven decision process and measures team productivity as the combination of collaborative effectiveness and the quality of creative flow. Both concepts are very broad (and possibly too abstract) and can only be comprehended using proxy metrics such as build speed, build and test reliability, debug time, and dependency issues. Thus, a DPE initiative focuses on tracking important metrics and monitoring.

There are currently no standard metrics for DPE, but four elements stand out as key pillars for optimizing software development processes from which metrics can be derived:

- **Degree of automation:** What is the proportion of tasks that are automated? What actions are developers required to perform manually?
- **Speed of cycles (lead times):** How much time are developers spending at each process stage (design, code, build, test, and so on)?
- **Correctness of the feedback loop:** Are errors clear and accurate (no false positives or false negatives) and discovered as early as possible so they don't impact the speed of the cycle?
- **Retrieval of information:** How much time do developers spend to obtain information? How do they usually proceed (documentation, messaging, meetings, and so on)?

Then, based on these metrics, DPE aims to contribute positively to the ROI (in terms of developer time and cost) for software development improvement efforts. It achieves these improvements through a variety of methods, including the following:

- Efficient tooling and infrastructure
- Automation and CI/CD
- Performance optimization in the build process
- Effective collaboration and communication
- Developer empowerment and reliable support

![DPE's four pillars and methods](https://i.imgur.com/xAwRAqu.png)

## Backstage as Part of Developer Productivity Engineering

Now that you have a solid understanding of what DPE is and the metrics you need to measure, let's look at how well Backstage and DPE can work together.

Does Backstage offer an effective way of achieving key DPE objectives? Can we use Backstage to improve developers' quality of creative flow and collaborative effectiveness?

### How Can Backstage Contribute to DPE Initiatives?

Companies should provide developers with resources, documentation, and hands-on guidance to help them navigate through the system and troubleshoot issues more efficiently. This is one of the key offerings of Backstage, which provides a [single pane of glass](https://www.techtarget.com/searchdatacenter/definition/single-pane-of-glass), including a [software catalog](https://backstage.io/docs/features/software-catalog/), [documentation](https://backstage.io/docs/features/techdocs/), and integration with other tools via plugins. In terms of DPE, this contributes to the key goal of optimal information retrieval.

![Backstage home page](https://i.imgur.com/LJEsNaE.png)
[*Image courtesy of Spotify*](https://backstage.io/blog/2020/03/16/announcing-backstage/)

Backstage is designed to be fully customizable and extensible. This means that each Backstage instance can be unique and tailored to meet your company's specific needs. You can find several plugins that help centralize the information in Backstage. For instance:

- The [API Documentation](https://github.com/backstage/backstage/blob/master/plugins/api-docs/README.md) and [GraphiQL](https://github.com/backstage/backstage/tree/master/plugins/graphiql) plugins provide a central place for API documentation, making it easy to work with APIs written by other developers.
- Most popular CI/CD tools have dedicated plugins, such as [GitLab](https://github.com/immobiliare/backstage-plugin-gitlab), [GitHub Actions](https://github.com/backstage/backstage/tree/master/plugins/github-actions), and [CircleCI](https://github.com/backstage/backstage/tree/master/plugins/circleci). These plugins can help developers navigate through the state of their application delivery process with back and forth between multiple UIs.

Another core feature of Backstage that can greatly contribute to DPE initiatives is [software templates](https://backstage.io/docs/features/software-templates/). Software templates automate workflows, helping developers save time and maintain project consistency by bootstrapping code repositories or provisioning infrastructure. Starting a new project can be time-consuming, and your company will greatly benefit from having all projects follow the same structure. Additionally, developers can be immediately productive if they change projects.

Backstage improves the developer experience through the software catalog's ability to display all relevant information in a single user interface and software templates that enhance automation capabilities. This is a great tool for DPE that empowers developers and makes them more autonomous in their day-to-day tasks.

### What Are the Limitations of Backstage DPE Initiatives?

Backstage can be a valuable tool for centralizing and managing various aspects of DPE monitoring and tracking metrics for your services and application delivery. For example, the [CI/CD Statistics plugin](https://github.com/backstage/backstage/tree/master/plugins/cicd-statistics) is a very useful tool. However, if you want to use this plugin, you have to write your own backend plugin to obtain the metrics.

Furthermore, Backstage can be complex, especially for companies and teams that are new to it. You only truly gain the benefits of Backstage once you have a dedicated team for it, as Backstage needs to be deployed in your own infrastructure, and you also need to perform maintenance, upgrades, and customization. You also need to implement a process to help with its adoption, as discussed in the [previous installment](URL) of this series.

Backstage is the perfect tool to build an internal developer platform, but the plugin ecosystem is still in development. Currently, there are no specific plugins that truly focus on DPE initiatives and provide insight into the speed of the cycle and the correctness of the feedback loop. In short, you won't be able to use Backstage to pilot your DPE initiatives and will have to rely on other tools for metrics.

Backstage only covers a small part of a DPE initiative. For instance, it won't help with performance optimization of processes and pipelines, which is a core element of DPE. Therefore, while Backstage is a really good product to improve the developer experience, if you don't have the bandwidth to allocate to this initiative, you might want to explore other opportunities with better ROI.

That being said, a well-planned Backstage initiative can easily overcome these issues. But, you will absolutely need to form a dedicated team with the skills to operate and maintain Backstage as well as [write custom plugins](URL). Then, ensure you have a clear plan on how you will encourage developers to adopt Backstage by making it a tool they understand and enjoy working with. If you don’t have the resources to fully commit to Backstage in these ways it will be very difficult to succeed.

## Conclusion

Developer productivity engineering (DPE) seeks to empower developers by simplifying workflows and enhancing collaboration so that they can spend most of their time designing and writing software. Backstage is a powerful tool for anyone looking to build a successful DPE initiative. By offering a centralized platform for cataloging your software ecosystem and providing seamless integration with your tech stack, Backstage demonstrates the potential to help your DPE initiatives. However, Backstage itself requires you to dedicate resources to add the features you need, either by installing existing plugins or creating your own.

You’ve now reached the end of the "Succeeding with Backstage" series. If you’ve followed along since the first series focused on evaluating Backstage, you should have all the necessary knowledge to implement Backstage and ensure its successful continued adoption. If you’ve only joined for this series, it might be worth checking out [Evaluating Backstage](https://www.kosli.com/blog/evaluating-backstage-why-backstage/) and [Implementing Backstage](https://www.kosli.com/blog/implementing-backstage-getting-started/) for a more in-depth look at the platform and its inner workings.

A final note: remember that one of Backstage's greatest strengths lies in its community. Now that you’re a Backstage pro, it might be worth considering how you can contribute to the community and cultivate even further improvements in the platform.


---
title: Evaluating Backstage Part 1: Why Backstage?
authors:
  - Carlos Eduardo Inocencio Alanis
---
This article is the first part of the "Evaluating Backstage" series. It covers all the basics around developer portals, introduces Backstage development, and explores how it can help your organization to be more efficient and secure.

Backstage is a developer portal that acts as a centralized hub for your organization, providing access to documentation, infrastructure, tooling, and code standards. It gives developers everything they need to create and manage their projects in a consistent and standardized manner. To understand why Backstage exists, and why you should care about it at all, you must first understand the pain points it's trying to alleviate.

Picture this: you're working on a project that needs to talk to some service, but no one is entirely sure who owns it anymore. Asking around only leads you to out-of-date documentation, and eventually you realize the code owner left the company months ago. You reverse engineer enough to identify the necessary infrastructure changes and raise a ticket. It takes days to get a response, only for you to find out that some crucial data is missing. And now your manager is asking when your project will be ready.

Stories like this are all too common in the developer world and are the inevitable result of growing organizations. There is no magic bullet for these issues, but developer portals such as Backstage can help to alleviate them.

![Example layout of Backstage developer portal](https://i.imgur.com/GbVl0mZ.png)

With a developer portal, individual team members no longer have to solve every challenge they face on their own because the portal enables better collaboration and communication across the whole team. For example, the portal allows administrators to define and enforce standardized practices for all projects. This way, developers can focus on problem-solving without having to worry about managing processes and adhering to different standards. To give a second example, if a new developer joins the organization and needs to create a simple function, the portal guides them on best practices, monitoring tools, code standardization, and accessing necessary resources like database secrets.

## What Is Backstage?

[Backstage](https://backstage.io/docs/overview/what-is-backstage) is an open source developer portal originally created by Spotify for its internal use. It was designed to allow their developers to maintain the [highly dynamic and autonomous](https://blog.crisp.se/wp-content/uploads/2012/11/SpotifyScaling.pdf) development flow Spotify has become known for. Backstage does this by helping you to organize and manage all the software your company produces. Recounting the entire history of Backstage is beyond the scope of this article, but it has been extensively [documented](https://engineering.atspotify.com/2020/04/how-we-use-backstage-at-spotify/) in [other places](https://backstage.io/docs/overview/background/). The tl;dr is that Spotify donated its developer portal to the Cloud Native Computing Foundation in 2020, making it open source. It's designed from the ground up to be flexible and allow every organization to adapt it to their own needs. In Spotify's own words:

> "It's a developer portal powered by a centralized software catalog—with a plugin architecture that makes it endlessly extensible and customizable."

Backstage provides a central place where you can find a registry of all deployed services, APIs, and infrastructure, as well as their documentation, and see how they interact with each other.

You no longer have to wonder who owns specific pieces of code, ask around only to be directed to out-of-date Confluence pages, or find that the dev you need to speak to no longer works at the company. You can view all your components in one central space and easily see which components are dependent on one another. Additionally, you'll have access to the source code's location and the name of its owner.

![Example of Backstage-generated documentation](https://i.imgur.com/OybFRMJ.png)

All of that is great by itself, but what really makes Backstage shine is another of its core functionalities: software templating. The core component of Backstage that deals with this is called Scaffolder. It provides a series of guidelines and restrictions that allow you to create templates that developers can use to deploy code. In these templates, you can bake in whatever your organization deems valuable for every type of component you want to create.

For example, you might want every serverless function to incorporate a specific monitoring tool, or you may want to ensure that developers don't forget to include a README file in their repositories. With software templates, developers can simply fill out a form and follow a few clicks to create their projects. The templates will automatically enforce the guidelines you defined, publish the repository to the appropriate workspace, and register the project in the software catalog.

![Example template catalog page inside Backstage](https://i.imgur.com/OuT2tON.png)

In the example provided earlier, this means you could create a template for the new developer to use, and they would simply log in to Backstage using your organization's LDAP. Once in there, they would locate the relevant template and provide it with all the data it requests. Backstage would then create a new GitHub repository that contains all the base code you usually need for such cases: layers, libraries, the CI/CD pipeline configuration file, automatically generated documentation, and so on, all within minutes.

![Example of component template in Backstage](https://i.imgur.com/62Vv29t.png)

Additionally, as time passes, a large open source community continues to grow behind Backstage. There are plenty of [plugins](https://backstage.io/plugins/) that make it easier to have your first working proof-of-concept app by providing open source solutions to common scenarios like resource visualization or cost reports.

This kind of flexibility has made Backstage a very attractive tool for many [high-profile companies and startups alike](https://github.com/backstage/backstage/blob/master/ADOPTERS.md), like Expedia, CVS Health, Siemens, LinkedIn, and Fiverr. Netflix also uses Backstage for its internal operations. If you would like to take a more in-depth look at how companies use Backstage, you can explore the [Expedia example](https://www.youtube.com/watch?v=rRphwXeq33Q&t=1508s).

## Adopting Backstage

This all sounds really good so far, right? Well, it's not as easy as it seems. If you have been reading about Backstage, you might have already come across different tutorials on how to make a simple application run locally and how to install plugins created by other people. Getting started is pretty straightforward. However, as soon as you start to move away from the out-of-the-box functions and try to implement more custom use cases, it starts to become a bit (or a lot) more confusing.

Backstage is not just an application or [SaaS](https://en.wikipedia.org/wiki/Software_as_a_service) that you can purchase and expect to work right out of the box. You have to think of Backstage as more like a framework than a fully-fledged application to implement in your organization. To implement it successfully, you will have to treat it like a product that requires support and an active team of developers. It's a project that will need your team's full engagement. 

You have to write code, modify the existing app code, and make it yours so it adapts to your use case—but at the same time, this is what makes Backstage powerful. As you progress, it will become yours and be tailored exactly to what your organization needs. All the effort put into it, plus the fact that no one knows your processes better than you do, means that it will tackle exactly the pain points you care most about. You can then contribute back to the [open source project](https://github.com/backstage/backstage) and help grow its ecosystem even more.

When adopting Backstage, it's also very important to think beyond technical details and code. As a developer, it's natural to get excited by the framework's endless possibilities and start going wild, implementing every impressive feature you can think of. However, it's essential to remember that there is little to no value in creating an awesome, shiny tool that no one will ever use because the user experience is bad. The portal must align seamlessly with the current workflow and be easily understandable. In a sense, you must do internal marketing and evangelize other teams about its usefulness if you want them to buy in. This means you need to think about *their* needs, user experience, and goals when you are building your portal. 

Become familiar with Backstage, read the documentation, read some articles explaining how to do things you would like to do - we will have lots of ideas to share with you as the series develops - and define specific metrics of success from the beginning. These will be useful for how you'll judge if Backstage is working well for your organization. Metrics like adoption rate, enhanced productivity, and automated processes will be specific to your organization, but measuring these will provide insights to guide you and keep you focused on what really matters to you. They will also make it easier to explain to management the added value you are bringing to the organization with this Backstage thing you keep talking about.

## Conclusion

In this introductory article, you've learned about the basics of developer portals and Backstage as a modular open source framework that allows extensions with your own or third-party plugins. On top of all this, if you've already dipped into the Backstage documentation, you might be feeling overwhelmed. The documentation sometimes assumes you already know what you're doing and can be complicated for Backstage newcomers. 

Don't be too hard on yourself if you don't get it right away. The rest of this series will break down the various elements of Backstage to make them a bit easier to digest. As you follow along, everything will start making sense, and you'll soon be contributing to the Backstage community yourself and helping to enhance this great framework. 

If this article has piqued your interest, you can continue your journey with the second installment, [Evaluating Backstage: How Backstage Works](https://www.kosli.com/blog/evaluating-backstage-how-backstage-works/).

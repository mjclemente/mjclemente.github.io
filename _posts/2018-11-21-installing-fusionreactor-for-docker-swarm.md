---
published: true
title: Setting Up FusionReactor for Docker Swarm (Part 1, Installation)
layout: post
tags: [lucee, docker, fusionreactor, cfml, coldfusion]
---
Moving a standard ColdFusion installation to Docker Swarm requires rethinking - and frequently rewiring - portions of our infrastructure. Today, that means reconfiguring how we deploy FusionReactor to monitor our applications.
<!--more-->

If you're already using FusionReactor, you know its value (and if you're not using it, you probably should be).[^1] Moving to Swarm means that you can't continue to use the traditional on-premise installation of FusionReactor; it's not suited to the distributed, dynamic environment of containers. Fortunately, there's an alternative: [FusionReactor Cloud](https://www.fusion-reactor.com/fusionreactor-cloud/).

In this post, I'll outline one approach to deploying your applications with FusionReactor Cloud.
Here's what's discussed:

* TOC
{:toc}

Two notes; first, this post assumes a degree of experience with Docker Swarm and CommandBox - learning to configure FusionReactor to monitor CFML apps on Docker Swarm isn't of much use unless you've already worked out the process of deploying those apps in the first place. Second, there are a lot of moving pieces here, so if I gloss over details you think are important, please let me know and I'd be happy to clarify.

## FusionReactor Cloud

Cloud is an extension of the standard FusionReactor installation.[^2] While its interface differs considerably from the on-premise version, the functionality tracks pretty closely. I'll review some of the differences in a future post. For our purposes, its most significant feature is that **FusionReactor Cloud supports dynamic server monitoring.** That is, you can use it for monitoring CFML apps running in Docker containers.

Before we get into how to use it, let's address how to get it, and what it costs.

## Pricing and Licenses

There's no reference to Docker or containers in the Cloud licensing section of the  [FusionReactor pricing page](https://www.fusion-reactor.com/pricing/), which is confusing. Here's what I found: *FusionReactor Cloud's current fair usage model allows/supports monitoring up to 5 container instances per license.*[^3]

Because the ratio of nodes-to-containers can vary considerably from one company to the next, this model may or may not work for your Swarm setup. If that's the case, I'll pass along the advice that I was given: **Reach out to the FusionReactor team to discuss your situation and Docker configuration.** They're a developer friendly organization, willing to help users transition to the cloud.

The rest of the steps here assume that you have a Cloud license available. If you don't, you can [register for a free trial](https://app.fusionreactor.io/auth/register).

## Installation

We'll be using the [CommandBox Docker image](https://hub.docker.com/r/ortussolutions/commandbox/); it makes configuration and deployment very simple. If you're not familiar with [CommandBox](https://commandbox.ortusbooks.com/), I encourage you to take a look; it's a remarkable tool for CFML development. Also, the rest of this post won't make sense if you're not familiar with how CommandBox can be used to spin up ColdFusion servers.

There's a [FusionReactor module](https://forgebox.io/view/commandbox-fusionreactor) on ForgeBox. When added as a dependency in an application's `box.json`, it installs FusionReactor for the server.

```json
{
  "name": "your.application.name",
  "version": "0.0.0",
  "dependencies":{
    "commandbox-fusionreactor" : "2.4.15"
  }
}
```

There are two approaches to ensuring that this module is installed and ready when your containerized CommandBox server starts:

* The first is to set the environment variable `BOX_INSTALL` to `true` in your Docker stack file, which triggers the `box install` command to be run before the server is started.
* The second option is use a custom Dockerfile, building from the Ortus image. In the classic tradeoff, this approach is more work, but it makes fine-grained customization of the image possible. When doing this, you need to include `RUN box install` prior to the server warmup and start, to ensure the dependencies are installed.

## Configuration

The FusionReactor module and its settings have a [dedicated page](https://commandbox.ortusbooks.com/embedded-server/fusionreactor) within the CommandBox documentation. Thanks to CommandBox, we can configure the module within our `server.json`, like so:

```json
{
    "name": "server.name.here",
    "othersettings": "etc. etc. etc.",
    "jvm": {
        "args": "-Dfrstartupdelay=30000 -Dfr.odl.activation.retry_amount=10 -Dfr.odl.activation.retry_interval=10000 -Dfrlicenseservice.logMessages=true"
    },
    "fusionreactor": {
      "enable": true,
      "licenseKey": "${FR_LICENSEKEY}",
      "licenseDeactivateOnShutdown": true,
      "cloudGroup": "tags,for,organization"
    }
}
```

___
Update (12/14/2018): The `jvm` key and `args` were added, following a discussion with FusionReactor support. I explain them [in this followup post](2018-12-14-update-to-fusionreactor-cloud-configuration-on-swarm.md).

___

Here's a breakdown of the settings used above, beneath the `fusionreactor` key:

* **enable**: Pretty self explanatory here. If this is set to `false`, FusionReactor functionality is disabled. This might be something you'd want to do for testing or in different environments.
* **licenseKey**: I'm passing this in via an environment variable set in my stack file. For Docker deployments, this is your FusionReactor Cloud license. In production, you'd want to set the environment variable via a Docker secret.
* **licenseDeactivateOnShutdown**: This maps to the FusionReactor Java property[^4] `frlicenseservice.deactivateOnShutdown`. When set to `true`, the instance being monitored will deactivate its license on shutdown. Obviously, that's what you want for a containerized server.
* ~~**licenseLeaseTimeout**: Mapped to the Java property `frlicenseservice.leasetime.hint`. This sets the number of minutes of inactivity before the license is released. Note that the minimum value is 10.~~

  See the [update to this post](2018-12-14-update-to-fusionreactor-cloud-configuration-on-swarm.md) for more on why *licenseLeaseTimeout* is not needed for Cloud deployments.
* **cloudGroup**: Mapped to the `fr.cloud.group` Java property. These group names are really helpful in organizing your reporting and alerts within FusionReactor Cloud. They're effectively tags for organizing your servers.

Assuming that the rest of your CFML stack is in order, with your `box.json` and `server.json` configured, you're ready to deploy to Swarm. The resulting CFML containers will be monitored within the FusionReactor Cloud portal and their licenses will automatically be deactivated as the containers are replaced.

### Update (12/14/2018)

A discussion with the FusionReactor support team lead to a [followup post](2018-12-14-update-to-fusionreactor-cloud-configuration-on-swarm.md) with some clarification and fine-tuning to aspects of this configuration.

## Some Final Notes

Getting FusionReactor installed in your containers and deployed to Swarm is half the process; the second half is working through the differences between the on-premise installation and FusionReactor Cloud. In my next post, I plan on examining some of those differences and discussing the process of setting up FR Cloud alerts that mirror the on-premise functionality. Until then, good luck Dockering!

___
[^1]: You don't have to take my word for it. Even though it's from 2015, [Ben Nadel's post about FusionReactor](https://www.bennadel.com/blog/2866-fusionreactor-offers-excellent-insight-into-java-and-coldfusion-server-performance.htm) does a great job of describing how it provides a wealth of information that was previously inaccessible. Ray Camden also has a post in which he [outlines the benefits of using FusionReactor](https://www.raymondcamden.com/2017/04/12/fusionreactor-still-the-best-for-coldfusion). And finally, if you hop on the CFML Slack channel to ask for help dealing with slow code, memory leaks, or a host of other unexpected behaviors, the first question you're going to get in response is whether you've installed FusionReactor.
[^2]: It's important to be aware that FusionReactor Cloud was not made exclusively or specifically for Docker deployments. When some aspect of its pricing or functionality seem strange, this is frequently the reason. It wasn't necessary made with the container use-case in mind. That said, the FR team is very responsive to user feedback, so let them know what you think and how you're using the product.
[^3]: Docker container pricing and other aspects of the Cloud offering are discussed in [this webinar](https://www.fusion-reactor.com/webinar/fusionreactor-cloud/) from early 2017.
[^4]: Documentation for the FusionReactor Java properties can be found here: [Using FusionReactor in Docker](https://docs.fusion-reactor.com/display/FR74/Using+FusionReactor+in+Docker).
---
published: true
title: FusionReactor for Docker Swarm (Part 2, Alerting)
layout: post
tags: [lucee, docker, fusionreactor, cfml, coldfusion]
---
I blogged, in two [earlier](/2018/11/21/installing-fusionreactor-for-docker-swarm.html) [posts](/2018/12/14/update-to-fusionreactor-cloud-configuration-on-swarm.html), about installing and deploying FusionReactor Cloud to monitor CFML applications on Docker Swarm. The next step is to to configure its updated alerting system to let you know if anything is amiss.
<!--more-->

The following is a not-exactly-brief guide to FusionReactor Cloud Alerting. While my use case is Docker Swarm, much of this is equally applicable to non-containerized applications and servers. I originally planned to include the process of migrating on-premise alerts to FusionReactor Cloud. However, this article grew a bit long, so that will need to wait for next post.

Without further ado, let's get familiar with the robust alerting interface that FusionReactor Cloud provides.

## Basics of Alerting in FusionReactor Cloud

*Alerting* is one of the primary three tabs within the [FusionReactor Cloud portal](https://app.fusionreactor.io/){:target="_blank"}, with sub-tabs for *Alerts*, *Checks*, and *Subscriptions*.

![FusionReactor Cloud Alert Pane][fr-cloud-alerting-pane]

We'll need to walk through each of these panes in order to configure our alerts. Here's the short breakdown of what they are and how they're related:

- **Subscriptions** are the *who* and *how* of Cloud alerts - that is, *who* should get alerted and *how* should they receive the alert? Subscriptions also provide control over *when* - you can limit them to certain days and/or times. To be of any use, a Subscription must be attached to a Check.
- **Checks** refer to the circumstances necessary to trigger an Alert. Broadly speaking, a Check is tied to either the status of a server or a defined set of data points (Memory Usage, Response Time, etc). Subscriptions are attached to these Checks and triggered when the threshold or status is met. These are the *why* - as in, *why* should an Alert be sent.
- **Alerts**, finally, are the *what*. Alerts are the consequence of Checks and Subscriptions - when a Check triggers a Subscription, an Alert is sent and logged.

So, taking them one at a time, in a little more depth:

### Subscriptions

As you'll see, FusionReactor Cloud offers far more granular control than the on-premise edition - this comes at the cost of significantly more configuration necessary to get it up and running.

In order to set up a Subscription, you'll first need to enable one of the many available integrations. An integration is a way of contacting someone; options include Slack, PagerDuty, custom webhooks, and more. We'll just stick with Email, as it's the easiest and most obvious choice.

Now, I'm of the opinion that the Email integration should be enabled by default, but it's not. So, once you're logged into your account, click the Account button in the upper right, and select the Configuration option.

![FusionReactor Cloud Configuration Menu][fr-cloud-configuration-menu]

Click the Configure button for Email, then all you need to do is choose the Save option... and the Email integration is enabled.

![FusionReactor Cloud Enable Email Integration][fr-cloud-email-configuration]

On to the actual Subscription setup. Proceed to the Alerting tab and the Subscriptions sub-menu. In the upper right, the **+Subscription** will open a tab for actually setting up who gets alerts (and when it happens).

![FusionReactor Cloud Subscription Tab][fr-cloud-subscription-tab]

Here, you start to see the level of control that Alerting in FusionReactor Cloud provides. You can restrict this Subscription to certain days of the week, or hours of the day. This may be helpful, for example, if you have a separate team for after-hours. Or, you can have separate Subscriptions for Warning vs. Error states - the latter being set to higher priority, or sent via a different integration.

We're setting up a basic email subscription, so we'll name it "Web Team Email Alerts", or something comparable, and choose the Email Service.

![FusionReactor Cloud Subscription Setup Step 1][fr-cloud-subscription-1]

The default subject provided, "FusionReactor alert", is fine; in the actual alert email this will be followed by the name of the Check that was triggered. Finally, we'll add the email address we want to send the alert to, and click "Save".

![FusionReactor Cloud Subscription Setup Step 2][fr-cloud-subscription-2]

Your Subscription is now saved; you'll see its details listed in the main Subscriptions panel, with the ability to duplicate/edit/delete, as well as test it. It hasn't been linked to any Checks yet, so let's do that next.

### Checks

In the Alerting section, within the Checks sub-menu, click **+Check** in the upper right to open the tab for building out a new Check.

![FusionReactor Cloud Checks Tab][fr-cloud-checks-tab]

Within this panel there's a lot going on. I'll walk through the options one step at time, but we'll saving actually creating a Check for the next blog post.

1. There are two types of Checks, *Threshold* and *Status*.

    ![FusionReactor Cloud Setup Check Step 1][fr-cloud-checks-setup-1]

    **Status checks** are the simpler of the two - they are triggered if a server or group of servers goes offline for a period of time.

    **Threshold checks**, as explained [in the documentation](https://docs.fusionreactor.io/guides/alerting/#checks), "are used to alert when a metric value crosses a defined threshold." Basically, FusionReactor is continually monitoring hundreds of metrics from your servers/applications (CPU Usage, Active Requests, etc.). Threshold checks enable you to select one of these metrics and, based on a percentage or frequency of its occurrence, trigger an Alert.

    Note that the *Name* entered here for the Check will be included in Subscription notifications that get sent, so it should be descriptive and clear. Optionally, you can also fill in a *Description*, for internal reference.

2. The Check is further refined by selecting the type of entity it should be applied to: *Server Instance*, *Group*, or *Application*.

    ![FusionReactor Cloud Setup Check Step 2][fr-cloud-checks-setup-2]

    **Server Instance**: Could be a physical server, VM, or a container instance. If you've got three container replicas for a Swarm service, each is a server instance. Generally, for Swarm deployments, you won't be choosing this option.

    **Group**: Server instances can be added to one or more groups - this is done on the startup of the server, via Java properties passed to FusionReactor. The FusionReactor module for CommandBox [makes these easy to set](/2018/11/21/installing-fusionreactor-for-docker-swarm.html#configuration), via `fusionreactor.cloudGroup` in `server.json`. I've found Groups very helpful when deploying to Swarm; they make it easy to monitor  multiple container replicas as a single entity.

    **Application**: The name says it; your applications are listed here. Even if the application is replicated across nodes, you can apply a Check to it.

3. Checks are built by first selecting a metric you'd like monitored and then defining a threshold at which you'd consider it an error - for example, if the metric Memory Usage exceeds the threshold of 80%. Three additional selectors, formatted as a sentence, can then be used to further specify when the Check should be triggered.

    ![FusionReactor Cloud Setup Check Step 3][fr-cloud-checks-setup-3]

    This "sentence" format is meant to be an intuitive way to define your Checks. Let's tackle its options in reverse order:

    - **Check timeframe**: The Cloud alerting engine runs once every 60 seconds. In order to allow time for ingesting/synchronizing data, the smallest window you can select here is 5 minutes. This is the timeframe within which FusionReactor monitors your error threshold, as well as the period of time it will take for any erroring Checks to return to an OK state. While I'm sure longer timeframes have their uses, I want my error notifications to be as close to realtime as possible, so I've exclusively used the "5 minutes" option here.
    - **Greater/Less Than**: Set the error state as greater or less than the threshold.
    - **Data point type**: There are four possible options here, three of which are self-explanatory: *single*, *all*, and *average*. I found the *count of* option to be less intuitive; it brings up an additional option displayed as ( 1/5 ), where you select the first number.

        ![FusionReactor Count Of Alert Syntax][fr-cloud-count-of-alert]

      The second number, I found out, refers to the timeframe of your check. Because the engine runs every minute, within a 5 minute timeframe there are 5 data points (so longer timeframes increase the second number). Selecting 1/5 would be the same as *single*, and selecting 5/5 is the same as *all* - the *count of* option gives you the ability to choose everything in between.

4. Pick the subscription(s) that this check should trigger. Checks can trigger more than one subscription alert.

    ![FusionReactor Cloud Setup Check Step 4][fr-cloud-checks-setup-4]

5. Finally, a preview graph is provided underneath, illustrating how your Check compares to the metric historically, so that you can see when it would be triggered.

    ![FusionReactor Cloud Setup Check Step 5][fr-cloud-checks-setup-5]

After clicking Save to add the Check, you'll see its details listed in the main Checks panel. As with Subscriptions, from here you can duplicate, edit, or delete it. You can also temporarily disable Checks from this page.

### Alerts

Finally, we'll move on to Alerts; fortunately, there's nothing we need to configure here. The Alerts panel provides logged reporting of your Checks; any time that a Check changes status, it's recorded here.

![FusionReactor Cloud Alerts Panel][fr-cloud-alerts-panel]

Basic logging here includes when Checks are added, paused, or deleted. These changes are recorded, but do not trigger Subscriptions. When a Check moves between the OK, WARNING, and ERROR statuses, the Subscriptions that are triggered will also be recorded here. The "View" option will open a panel with more information about the Alert/Check/Subscription.

## For Next Time

As mentioned at the beginning, I had hoped to discuss the process of migrating FusionReactor's on-premise Crash Protection alerts to the Cloud alerting interface - because of their differences, it's not exactly straightforward or intuitive. That was actually the original motivation for writing this piece, but I had to get all the initial configuration out of the way, which proved to be a bit more writing than anticipated.

So, in the next post in this series we'll build on what we learned here and set up the FusionReactor Cloud equivalents of the on-premise Quantity, Runtime, and Memory Crash Protection alerts.

Happy New Year!

[fr-standard-alerts]: /public/assets/images/fusion-reactor-standard-protection-alerts.png
[fr-cloud-alerting-pane]: /public/assets/images/fusionreactor-cloud-alerting-tab.png
[fr-cloud-configuration-menu]: /public/assets/images/fusionreactor-cloud-configuration-menu.png
[fr-cloud-email-configuration]: /public/assets/images/fusionreactor-configured-email-alert-service.png
[fr-cloud-subscription-tab]: /public/assets/images/fusionreactor-add-subscription-tab.png
[fr-cloud-subscription-1]: /public/assets/images/fusionreactor-subscription-step-1.png
[fr-cloud-subscription-2]: /public/assets/images/fusionreactor-subscription-step-2.png
[fr-cloud-checks-tab]: /public/assets/images/fusionreactor-add-check-tab.png
[fr-cloud-checks-setup-1]: /public/assets/images/fusionreactor-check-setup-step-1.png
[fr-cloud-checks-setup-2]: /public/assets/images/fusionreactor-check-setup-step-2.png
[fr-cloud-checks-setup-3]: /public/assets/images/fusionreactor-check-setup-step-3.png
[fr-cloud-checks-setup-4]: /public/assets/images/fusionreactor-check-setup-step-4.png
[fr-cloud-checks-setup-5]: /public/assets/images/fusionreactor-check-setup-step-5.png
[fr-cloud-alerts-panel]: /public/assets/images/fusionreactor-cloud-alerts-panel.png
[fr-cloud-count-of-alert]: /public/assets/images/fusionreactor-count-of-alert-setting.png
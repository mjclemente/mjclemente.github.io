---
published: true
title: FusionReactor for Docker Swarm (Part 3, Migrating On-Premise Alerts)
layout: post
tags: [lucee, docker, fusionreactor, cfml, coldfusion]
---
This was the post I initially set out to write when I started this series.
<!--more-->

Let's take a look at where we're starting from


## Standard On-Premise Alerts

Within an on-premise installation of FusionReactor, there are three standard protection alerts that you can configure:

- **Quantity**: When the number of running requests exceeds a set threshold.
- **Runtime**: If a process runs for longer than a specified number of seconds.
- **Memory**: When memory usage exceeds a selected percent for a period of time.

These are found within the "Protection" tab and are fairly simple to set up.

![Webprotection Alerts in FusionReactor Standard][fr-standard-alerts]

When I set out to reproduce these in FusionReactor Cloud, I found that it wasn't quite so straightforward.

But we're getting ahead of ourselves; let's get familiar with the robust alerting interface that FusionReactor Cloud provides.

## Migrating On-Premise Crash Protection Settings

With that overview of the alerting interface and configuration out of the way, let's move on (finally) to recreating the on-premise FusionReactor alerts. There are two requirements for these checks:

1. Make sure that you've set up an [email Subscription](#subscriptions) - it's what these Checks will trigger when they error.
2. For Swarm deployments, you'll need a Fusion Reactor server group that includes all your CFML containers in the Swarm. This will provide a global snapshot of the metrics we're monitoring.

### Request Quantity Check

As you'll recall, in the on-premise version, this protection alert is triggered

### Runtime Check

### Memory Check

### Bonus: Uptime Check

specify the conditions that should trigger alerts
___

[fr-standard-alerts]: /public/assets/images/fusion-reactor-standard-protection-alerts.png
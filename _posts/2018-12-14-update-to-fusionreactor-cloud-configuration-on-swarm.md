---
published: true
title: Updated Configuration for FusionReactor Cloud on Docker Swarm
layout: post
tags: [lucee, docker, fusionreactor, cfml, coldfusion]
---
Following my [previous post](/2018/11/21/installing-fusionreactor-for-docker-swarm.html) about setting up FusionReactor on Docker Swarm, we encountered an issue with deployments failing to report. After a dozen or so emails with the fantastic FusionReactor support team, everything is working again. Here's what I learned.
<!--more-->

## The Problem

During a routine deployment to Swarm, one of our FusionReactor server groups (3 containers) disappeared from the Cloud interface. There were no actual issues or errors with the application; it just stopped reporting.

When a FusionReactor engineer took a look at the log files, he determined the source of the problem was an issue with the license activation. As he explained it, if the network adapters take too long to come up, FusionReactor stops trying to activate, so the instances can't report.

## The Solution

This problem with activations, apparently, is a known issue that's slated to be fixed in the next major FusionReactor release in 2019. Until then, the workaround provided was to pass in additional -D properties[^1] to the server. They were:

- `-Dfrstartupdelay=30000`
- `-Dfr.odl.activation.retry_amount=10`
- `-Dfr.odl.activation.retry_interval=10000`
- `-Dfrlicenseservice.logMessages=true`

As you could probably figure out, these properties delay the initial startup for FusionReactor, increase the number of retry attempts for license activation, and increase the length of the retry interval. The final one adds some additional logging in order to make license debugging easier.[^2]

We can use CommandBox's ability to start up a server with [ad-hoc JVM args](https://commandbox.ortusbooks.com/embedded-server/configuring-your-server/jvm-args#ad-hoc-jvm-args) in order to apply these to our CommandBox Docker image. See [Configuration](#configuration) at the end for example of this.

## What Else I Learned

In my initial configuration for the CommandBox FusionReactor module, I included the  `licenseLeaseTimeout` setting in my `server.json`. This maps to the Java property `frlicenseservice.leasetime.hint`, which, as I understood it, "sets the number of minutes of inactivity before the license is released". I set this to 10 minutes, its lowest possible value, in order to free up licenses as containers were destroyed and redeployed.

Apparently, this isn't necessary. Over the course of my discussion with the FusionReactor engineer, I learned that the cloud and on-premise versions of FusionReactor use different licensing APIs - the lease timeout hint is only applicable to the on-premise version. **Cloud licenses are automatically freed up much faster - 3 minutes of offline time releases the cloud reservation for use by another instance.**

## Configuration

I updated my earlier post about [using FusionReactor Cloud with Swarm](/2018/11/21/installing-fusionreactor-for-docker-swarm.html) to reflect what I learned. Here, again, is the updated configuration for a `server.json`, in order to start it up with a Cloud license:

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
[^1]: Currently, these properties aren't publicly documented, but I was told they'd be added to the official documentation in the future.
[^2]: While the `-Dfrlicenseservice.logMessages=true` flag is not necessary, except for debugging problems, the logging it enables only occurs once daily and is minimal, so I don't see a downside in keeping it on, in the event that it's needed at a future time.
---
published: true
title: Install Lucee Extensions on CommandBox Docker Containers
layout: post
tags: [lucee, extensions, commandbox, docker, forgebox, coldfusion, cfml]
---
One of the issues you need to tackle when deploying Lucee containers is automating the installation of server extensions; there are a few methods, each with its own tradeoffs. To the current list of approaches, I'm adding [`docker-lex-install`](https://github.com/mjclemente/docker-lex-install), a CommandBox module I wrote to handle this process.
<!--more-->

## The Backstory: Using Memcached
At work, we had a CMFL app that we needed to containerize. Our goal was to deploy three replicas of the service across a three node Docker Swarm. The app uses sessions, so we needed to address the issue of session persistence within the cluster.

It's my understanding that using external session storage is preferable to using "sticky sessions" via the load balancer - external sessions can persist through server restarts (which is awesome!) and traffic can be round-robined and balanced more effectively. Lucee's built-in support for external session storage made choosing this route even easier.

Enabling external session storage in Lucee is done by creating a named, external cache and assigning it the role of session storage. [Memcached](https://www.memcached.org/), [Redis](https://redis.io/), and [Couchbase](https://www.couchbase.com/) are common names that come up in the discussion of caching solutions, and each can be used with Lucee.[^1] Memcached is the most basic of the three, but because the Memcached driver extension is free and provided by the Lucee Association, we decided to give it try.

All of which is a roundabout way to circle back to the initial problem - we needed a way to install the official Lucee Memcached server extension as part of our scripted CommandBox container deployment.

## Current Approaches to Extension Installation

As I understand it, there are three existing approaches for installing extensions when deploying Lucee containers:

1. __Custom CF Engine__: The first strategy, creating your own CF Engine and deploying it with CommandBox, is the most work and least flexible. By customizing it, you can preinstall any extensions you'd like. [^2]
2. __wget__: Because Lucee automatically installs extensions from the `lucee-server/deploy/` folder, it would be possible to set up your Dockerfile to install them. This approach is easier using the official Lucee Docker image than via CommandBox. Your Dockerfile command would look something like this:
```text
RUN wget -nv http://extension.lucee.org/rest/extension/provider/full/16FF9B13-C595-4FA7-B87DED467B7E61A0?version=3.0.2.29 -O /opt/lucee/server/lucee-server/deploy/extension-memcached-3.0.2.29.lex
```
3. __Environment variables__: As of Lucee 5, you can use environment variables to install extensions. This setup is explained by Justin Carter in his tutorial [
Installing the Memcached extension for Lucee 5.x ](https://labs.daemon.com.au/t/installing-the-memcached-extension-for-lucee-5-x/319). It's a really helpful guide, regardless of the extension you're installing - it's helpful even if you don't ultimately use this approach. The general syntax looks like this:
```text
ENV LUCEE_EXTENSIONS "16FF9B13-C595-4FA7-B87DED467B7E61A0;name=Memcached;version=3.0.2.29"
```

Note that if you use one of the latter two strategies, you also need to prewarm the server, so that it can recognize and install the extensions. Depending on your use case, you may any one of these approaches acceptable.

## Using the `docker-lex-install` CommandBox module
To the above list, we now add a fourth option: `docker-lex-install`. Along with the basic requirement of being scriptable, there were three primary motivations for building this module for installing Lucee server extensions. I wanted it to:

1. Minimize dependencies on external services (in this case, [http://extension.lucee.org](http://extension.lucee.org)).
2. Ensure that it worked easily with CommandBox deployments
3. Make sure installing multiple extensions was easy and straightforward.

It does these things well, I believe. As a CommandBox module, it's pretty powerful, and it's easy to use. Installation is as simple as declaring `docker-lex-install` as a project dependency. The module then automatically runs `onServerStart()`, and scans a specified folder for .lex files (i.e. Lucee EXtension). Any extensions found there are queued for installation by being moved to `lucee-server/deploy/`. At its simplest, adding an extension is as simple as dropping its .lex file in a folder.

There's obviously a little more to it than that. But not much. If you're interested in the specifics of using `docker-lex-install`, head over to [its Github repo](https://github.com/mjclemente/docker-lex-install) for instructions. This approach isn't for everyone; there's no one-size-fits-all solution for these types of concerns. Use what works for your organization's priorities. ; I think the README is pretty straightforward, but if you have any questions or issues, please let me know!

___
[^1]:For Redis and Couchbase, Ortus Solutions built Lucee extensions - the [Redis Lucee Extension](https://www.ortussolutions.com/products/redis-lucee) and [Couchbase Lucee Extension](https://www.ortussolutions.com/products/couchbase-lucee) respectively. Keep in mind that these are paid commercial products.
[^2]:I haven't actually done this, but as I understand it, you basically start up a server, manually customize all the settings, zip up the server home directory, and rename the extension to .war. Brad Wood provides more details about halfway through [this article](https://www.ortussolutions.com/blog/configuring-your-commandbox-servers-on-first-start).
---
published: true
title: Script to Create Docker Swarm on DigitalOcean
layout: post
tags: [docker, digitalocean, containers, container orchestration, doctl]
---
My goal here was to put together a script for easily setting up a Swarm on DigitalOcean. As with the [previous post](/2019/03/04/script-docker-host-creation-digitalocean-doctl.html), I wanted to do this without relying on Docker Machine, so again, I leaned on [`doctl`](https://github.com/digitalocean/doctl) to do the heavy lifting.
<!--more-->

TL;DR: [Get the Swarm Creation Script on GitHub](https://github.com/mjclemente/do-swarm-create)

The resulting script provides some basic configuration options (Droplet size, number of masters/workers, etc) and automates the Swarm creation process. It strikes a balance between the simplicity/drudgery of creating a Swarm manually and the complexity/power of using a tool like [Terraform](https://www.terraform.io/) to manage infrastructure.

There's not a lot more to say here, as I (hopefully) included the necessary documentation in the [README](https://github.com/mjclemente/do-swarm-create).

I don't know what kind of demand there is for a script like this, but at a minimum, it's helpful for presentations, demos, and folks getting started.

So, check it out, clone it, use it, fork it, and let me know if you have any questions or issues: [**mjclemente/do-swarm-create**](https://github.com/mjclemente/do-swarm-create)
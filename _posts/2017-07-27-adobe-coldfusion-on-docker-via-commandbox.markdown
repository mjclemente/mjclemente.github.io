---
published: true
title: A Complete Beginner Runs Adobe ColdFusion on Docker (Mac)
layout: post
tags: [docker,coldfusion]
---

This is the third in a series of posts about getting started with Docker. In the first I [set up Docker on my machine](/2017/07/25/getting-started-with-docker.html); the second covered [running ColdFusion on Docker (with Lucee)](/2017/07/26/getting-started-with-coldfusion-for-docker.html). My next step was to tackle Adobe ColdFusion (ACF), which turned out to be easier than anticipated.
<!--more-->

Problem: There's no official Adobe ColdFusion Docker image. 

I've heard, at various conferences and events, that Adobe will be releasing an official image sometime this year, but that doesn't help right now. I wasn't sure how best to proceed, so I hopped on the #docker channel in the [CFML Slack group](https://cfml-slack.herokuapp.com)[^1] to ask how people handle this. Almost immediately, [Eric Peterson](https://twitter.com/_elpete) pointed out that there's a [CommandBox](https://www.ortussolutions.com/products/commandbox) Docker image that enables you to spin up Adobe ColdFusion.[^2]. 

So, let's get started with the [CommandBox Docker image](https://github.com/Ortus-Solutions/docker-commandbox):

1. The first step is to clone the repository to your local machine. So, open up the terminal, and navigate to wherever you store projects or tools. Then run:

	<pre class="highlight">
	git clone git@github.com:Ortus-Solutions/docker-commandbox.git
	cd docker-commandbox</pre>

2. Now we need to edit the project's `docker-compose.yml`, so open that up in the code editor of your choice. 
3. We're going to change the file substantially, so rather than try to explain it, you can just take a look at [the end result below](/2017/07/27/adobe-coldfusion-on-docker-via-commandbox.html#updated-docker-composeyml). I explain the changes there. It's set up for ACF 11 and you'll need to map it to your local project.
4. Once you've made the changes outlined, just run the magical `docker-compose up` 
5. The ColdFusion 11 image will download (it takes a bit longer than Lucee) and once it completes, you'll be able to access your project at [http://127.0.0.1:8080/](http://127.0.0.1:8080/). I dumped the `server.coldfusion`, so this is what I saw:

![Server Scope Dump from Adobe ColdFusion 11 in Docker](/public/assets/images/acf-server-coldfusion-dump-on-docker-image.png)
6. That's it! You're running an instance of Adobe ColdFusion (Developer Edition) in Docker. Now let's take a look at the `docker-compose.yml` file that made this possible. 

## Updated docker-compose.yml
```yaml
version: '3.3'

services:

  web:
    image: ortussolutions/commandbox:latest
    ports:
      - "8080:8080"
    volumes:
      - /Users/matthew/www/projects/testing:/app
    environment:
      PORT: 8080
      CFENGINE: adobe@11
      cfconfig_adminPassword: admin123
```
First, here's the official [reference for the docker-compose.yml file](https://docs.docker.com/compose/compose-file/). 

### version 
`version: '3.3'`

Updated to the latest version. Not an absolutely necessary change, but because I have the latest version of Docker installed, I can use the latest Docker Compose file format. There's nothing in this file that necessitates updating this from the `2.1` version that is currently set in the CommandBox repository.

### image
`image: ortussolutions/commandbox:latest`

This is actually an important change. Setting the repository to use the latest version of the CommandBox image ensures that you have the latest stable version, with all available fixes. I had initially left this with the `3.6.0-snapshot` and it caused some issues. 

### ports
`- "8080:8080"`

Unchanged from the CommandBox repository. Simply map a port on your machine to a port in the container.

### volumes
`- /Users/matthew/www/projects/testing:/app`

You'll want to map this to a folder somewhere on your own machine. It doesn't matter where it is. As I pointed out when setting up Lucee in the previous post, the mapping of volumes, like the mapping of ports, is based on the`:`(colon) separator. Your local path is on the left, the image's path is on the right. 

### environment
These are passed into your container as environment variables. 

`PORT: 8080`

Let CommandBox know what port it should start the server on.

`CFENGINE: adobe@11`

For this post, this is the most important. It tells CommandBox that we want to run Adobe ColdFusion 11. You can find the [available CF Engines on ForgeBox.io](https://www.forgebox.io/type/cf-engines). Multiple major and minor releases of Adobe ColdFusion are available, as well as the Railo and Lucee server engines.

`cfconfig_adminPassword: admin123`

A handy feature to set the admin password for ColdFusion. If you don't set it, the password defaults to: *commandbox*. The username is: *admin*. This is set using a tool called [CFConfig](https://github.com/Ortus-Solutions/cfconfig), which I plan on exploring further.

## Wrap Up
Well, it was easier than anticipated to set up Adobe ColdFusion on Docker. That's not something you often hear.

There's obviously more that needs to be done in order to build this out into a proper development environment. In the coming weeks I want to explore what else we can do with CFConfig, working with databases, probably a few other issues that I haven't even considered yet.

<hr />
[^1]: Here's [Adobe's post about it](http://blogs.coldfusion.com/adobe-coldfusion-slack-channel/). If you're a ColdFusion developer, I cannot recommend it highly enough. The community is incredibly helpful.
[^2]: I haven't done a post on CommandBox, but again, if you're a ColdFusion developer, you really, really should be. It's hard to explain how helpful it is. If you've got questions about it, feel free to ask me, or, hit up [Brad Wood](https://twitter.com/bdw429s). It will make your life easier, I promise.

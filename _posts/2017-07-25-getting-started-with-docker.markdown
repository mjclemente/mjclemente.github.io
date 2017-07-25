---
published: true
title: A Complete Beginner Gets Started Using Docker (Mac)
layout: post
tags: [Docker,Lucee]
---

Our team is planning on moving to Docker in the coming months. After attending a number of Docker-related sessions at [cf.Objective()](http://www.cfobjective.com/) this past week, it seemed time to get my hands dirty and actually start working with it. The following is a painfully simple walkthrough of the steps I took to go from zero-to-Docker. 
<!--more-->

1. Everyone says to get started at [docker.io](https://www.docker.io), but that site redirects now. If you're a developer, start at [www.docker.com/get-docker](https://www.docker.com/get-docker); it's where you end up after navigating all the marketing on the homepage.
2. There are two editions of Docker available. We're just getting started, so we want the free, [Community Edition](https://www.docker.com/community-edition).
3. Following an overview of the Community Edition, I followed the link to [download Docker CE for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac) from the Docker Store.
4. I used the Stable download (as opposed to the Edge version) and ran through the installation of the desktop environment. It culminated with the cute message: "We are whaly happy to have you."
5. At this point the desktop app prompts you to "Sign in / Create a Docker ID". Since I didn't have a Docker ID set up yet, I followed the link to [cloud.docker.com](https://cloud.docker.com).
6. I set up my Docker ID, confirmed my email address, and logged in. 
7. I got a "Welcome to Docker Cloud" message, with some additional steps that I could take, and guides to Docker Concepts. I skipped these for now and went back to the desktop installation.
8. Now that I had my Docker ID, I logged into desktop app.
9. Then, following the installation instructions, I opened my terminal and ran `docker version`, which showed the version installed (it was *17.06.0-ce*, for the record).
10. I then ran `docker run hello-world`, which confirmed that images were being pulled successfully, and which also provided a helpful explanation of what actually happened during the command.
11. As a side note, I wondered where the images were being stored on my machine. A quick search lead me to [this answer](https://forums.docker.com/t/where-are-images-stored-on-mac-os-x/17165): 

```text
$HOME/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2
```
	
Admittedly, there's nothing particularly insightful or complicated here. However, sometimes the hardest part of learning/using a new technology is overcoming inertia and just getting started. Next, I'll be diving into getting the Lucee CFML engine up and running on my Docker setup. 
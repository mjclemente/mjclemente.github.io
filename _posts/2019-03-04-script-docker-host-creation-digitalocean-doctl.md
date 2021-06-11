---
date: 2019-03-04
published: true
title: Scripting Docker Host Creation on DigitalOcean without Docker Machine
layout: post
tags: [docker, digitalocean, containers, container orchestration, doctl]
---
Months ago I ambitiously began a series of posts about setting up and deploying a production Docker Swarm. Well, I *intended* it to be a series. I didn't actually get around to writing a second article until now, and for better or worse, this will basically serve as a revision of that first post, in which I documented [using Docker Machine to script host creation](/2018/07/15/create-docker-nodes-on-digitalocean-with-shell-script.html).

<!--more-->

 The process I outlined there still works, but I no longer think it's a particularly good approach. Now you may be thinking, why not? Read on...

___

* TOC
{:toc}
___

## Why Not Docker Machine?

**tldr;** There are just better options available.

*[Docker Machine](https://github.com/docker/machine) was never a perfect solution*. As I mentioned in the initial post, it wasn't designed for team use - sharing configuration and control of remote hosts couldn't be done easily. Three of the top-voted, unresolved enhancement requests on the repo have to do with this functionality.[^1] Additionally, around the time that first post was published, Docker Machine was placed in [maintenance mode](https://github.com/docker/machine/issues/4537). The project isn't abandoned -  it's still receiving commits and its latest release was in January of this year (2019). However, it's not a focus/priority for the Docker organization any more.

### Updates in Docker 18.09

In contrast with Docker Machine, the feature set of the Docker Engine and CLI is being actively developed and expanded. Of particular relevance to this post is functionality added in [version 18.09](https://docs.docker.com/engine/release-notes/#18090)[^2] from November 2018, making it significantly easier to run Docker commands on remote hosts. Setting the `DOCKER_HOST` environment variable enables you to switch the context of your Docker commands to run on remote machines. Said differently, you no longer need to SSH into servers in order to execute `docker` commands - you can operate local commands against a remote system. Here's how that looks:

```bash
export DOCKER_HOST=ssh://user@server
# docker commands will now be executed against the remote server

docker container ls
# lists containers running on the remote host

unset DOCKER_HOST
# docker commands will again be executed on your local system
```

This was one of the benefits of Docker Machine - the ability to easily manage remote Docker servers. Now it's part of the core Docker functionality - without the headaches and limitations of Docker Machine.

## Creating Docker Hosts with `doctl`

In my initial post I mentioned `doctl`, DigitalOcean’s official command-line client. At that time, I only utilized it to list configuration options available when creating new servers - it’s capable of so much more. I’ve since realized that `doctl` can be used to create Docker hosts on DigitalOcean, removing the need for Docker Machine.

This is made possible by DigitalOcean's [One-Click Docker Image](https://www.digitalocean.com/docs/one-clicks/docker/). As explained in its documentation, this image uses Ubuntu 18.04.1 and:

>  automatically installs and configures the open-source Docker CE (Community Edition) and Docker Compose according to the official Docker recommendations.

That sounds like an ideal setup, and we can use `doctl` to script Droplet creation with this image. Let's look at how that's done.

### Prerequisites 

Because this post serves as a revision of the [previous one](/2018/07/15/create-docker-nodes-on-digitalocean-with-shell-script.html#setting-up-a-digitalocean-account), the prerequisites for using this script are largely the same. Here’s a quick recap of what you'll need:

- A Digital Ocean account ([$100 promo with referral](https://m.do.co/c/8acbd6928587))
- SSH credentials added to your account ([DO Guide](https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/))
- A Personal Access Token for the DigialOcean API ([DO Guide](https://www.digitalocean.com/docs/api/create-personal-access-token/))

To this list, we'll be adding one more requirement.

#### Install `doctl`

You'll need to install and authenticate `doctl`. Four approaches to installation are provided on the project's Github repo, [here](https://github.com/digitalocean/doctl#installing-doctl). I'll leave the nitty-gritty of that up to you; once it's installed, be sure to follow their instructions for [Authenticating with DigitalOcean](https://github.com/digitalocean/doctl#authenticating-with-digitalocean). Finally, the [How To Use Doctl](https://www.digitalocean.com/community/tutorials/how-to-use-doctl-the-official-digitalocean-command-line-client) guide is an invaluable resource; bookmark it at the very least. 

### Configuration with Variables

In the previous version of this script, configuration was done via environment variables. That was definitely a bit overkill. In this case, the variables that control installation are included in the script, at the top. Here they are, with a brief explanation of their purpose and default values:

* **DO_DROPLET_NAME** 
  Sets the prefix for the Droplet name, which will be followed by incrementing numbers. I've set the default to `docker-node`, so if you scripted the creation of three servers, they would be named `docker-node-1`, `docker-node-2`, and `docker-node-3`, respectively. 
* **DO_SIZE** 
  Sets the size of the Droplets being created. You can get a list of available options by running `doctl compute size list`. The default value set is `s-1vcpu-1gb`, which is 1 GB / 1 CPU / 25 GB SSD disk. Currently [$5/mo](https://www.digitalocean.com/pricing/#Compute).
* **DO_REGION** 
  Sets the Droplet region; options can be seen by running `doctl compute region list`; defaults to `nyc1`.
* **DO_SSH_IDS**
  Provides a list of SSH keys that should be added to the Droplets being created. SSH keys can be referenced either via their DigitalOcean resource Id, or their fingerprint. By default I have this including all the SSH keys added to your account. These are provided via the script `doctl compute ssh-key list --no-header --format ID`. Alternatively, you could use the command `doctl compute ssh-key list` to retrieve your SSH keys and selectively add them to this variable.
* **DO_TAGS**
  Tags are helpful for organizing and managing Droplets. You can set tags here, comma separated. `demotag` is the only tag included by default.
* **DO_DROPLET_COUNT**
  Sets the number of Droplets to create. Defaults to `3`.

Okay, enough with the preparation, on to the action.

## Creating Docker Servers from the Command-line

Here’s the updated script; I’ll provide additional commentary and explanation below.

```shell
#!/bin/bash

DO_DROPLET_NAME=docker-node
DO_SIZE=s-1vcpu-1gb
DO_REGION=nyc1
DO_SSH_IDS=$(doctl compute ssh-key list --no-header --format ID)
DO_TAGS=demotag
DO_DROPLET_COUNT=3

: "${DO_SSH_IDS:?Please set your DO_SSH_IDS}"

for server in {1..$DO_DROPLET_COUNT}; do
  doctl compute droplet create $DO_DROPLET_NAME-${server} --size $DO_SIZE --image docker-18-04 --region $DO_REGION --ssh-keys $DO_SSH_IDS --tag-names $DO_TAGS --enable-backups --enable-monitoring --enable-private-networking --wait
done

unset DO_DROPLET_NAME
unset DO_SIZE
unset DO_REGION
unset DO_SSH_IDS
unset DO_TAGS
unset DO_DROPLET_COUNT
```
For convenience, here's a [ Gist of `create-docker-servers-doctl.sh`](https://gist.github.com/mjclemente/ada1a591bce5f4fedcd6d6d992d95705) that you can reference, fork, etc. 

This script may take a few minutes to run, but when it does complete, you'll have three brand new Droplets with Docker installed running in your DigitalOcean account. You can confirm they exist by running one of the following commands:

```bash
doctl compute droplet list
# Will list all your Droplets

doctl compute droplet list --tag-name demotag
# Limit the Droplets listed by tag 
```

You'll notice there are a few options set on this creation script. Here's a quick overview of what they are:

- `--enable-backups` A system-level disk image of the entire Droplet will be taken once a week and saved for four weeks. Adds 20% to the cost of the Droplet. You can read more about backups [here](https://www.digitalocean.com/docs/images/backups/overview/). 
- `--enable-monitoring` Enables the collection of metrics about the Droplet for reporting, monitoring, and alerts.
- `--enable-private-networking` Makes Droplet-to-Droplet networking available, within the same region.
- `—wait` Wait for the Droplet to be created.

It's also worth pointing out that the DigitalOcean One-Click Docker image enables the [UFW firewall](https://help.ubuntu.com/community/UFW) and locks down the available ports. While a good security practice, this can be confusing if you forget about it. Swarm mode, for example, won't work, unless you open up the [required ports and protocols](https://docs.docker.com/engine/swarm/swarm-tutorial/#open-protocols-and-ports-between-the-hosts).

## Removing the Droplets

If you want to delete the Droplets, you can also do this from the command-line using `doctl`. The following command will remove the Droplets created in this demo:

```bash
doctl compute droplet delete docker-node-1 docker-node-2 docker-node-3
```
Add `-f` to remove them without confirmation. 

## Conclusion

The conclusion is basically the same as it was last time. I hope to be moving on to Swarm creation and configuration. That *should* be the next topic in this series. Fingers crossed.

As always, one of my goals is to keep things simple; I want a process that works and that I understand. This space is changing rapidly, so I'd love to hear about other tooling and approaches.

___
[^1]:In case you're interested, the are [Export / import machines (#23)](https://github.com/docker/machine/issues/23), [Proposal: machine share (#179)](https://github.com/docker/machine/issues/179), and [Attach to existing machine from another client (#1328)](https://github.com/docker/machine/issues/1328). The issue is also addressed [here by Bret Fisher](https://github.com/BretFisher/ama/issues/19).
[^2]:For those interested in a little more insight on this release, check out [Bret Fisher's Youtube AMA](https://www.youtube.com/watch?v=5EuDY6ayNs8&feature=youtu.be&t=19) on it.
---
date: 2016-12-22
published: true
title: Why Does Git Keep Asking for My SSH Password (Bitbucket / Github)?
layout: post
tags: [git,ssh]
---
I'm making an effort, when I learn something while resolving an issue, to document the process. I'd rather not have to muddle through the haze of déjà vu trying to solve the same problem a second and third time. The writing both helps me learn and serves as a resource when I forget. This is one of those "documentation" posts; hopefully others find it helpful as well. <!--more-->

I recently updated to MacOS Sierra (10.12.2). After the update, git began prompting me for my ssh password on every interaction with the remote repo:

```shell-session
Enter passphrase for key '/Users/matthew/.ssh/id_rsa':
```
I started looking around for a solution online, and initially most of what I found had to do with people [cloning repos via https, instead of git](http://stackoverflow.com/questions/8600652/git-on-bitbucket-always-asked-for-password-even-after-uploading-my-public-ssh), which was not applicable.

Finally, [this Stack Overflow post](http://stackoverflow.com/questions/33017216/server-keeps-asking-for-password-after-i-added-my-key-to-bitbucket) pointed me in the right direction - I needed to make sure the SSH agent was running, and that my key was added to it. Spoiler - I just needed to re-add my key to the SSH agent to resolve the issue.

I ran the following command (after reading Github's [guide on SSH agent forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/)) to check if my key was available to the agent:

```shell-session
$ ssh-add -L
The agent has no identities.
```

This was obviously the issue; somehow my key had been forgotten. The guide goes on to explain that:

>On Mac OS X, ssh-agent will "forget" this key, once it gets restarted during reboots.

Frustrating, but easy enough to resolve. Because I couldn't remember how to add the key, I tracked down the Github guide on [setting up SSH keys](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/). First I had to make sure the agent was running:

```shell-session
$ eval "$(ssh-agent -s)"
Agent pid 7964
```
Then I added my key:

```shell-session
$ ssh-add ~/.ssh/id_rsa
```
And the issue was resolved. Running `ssh-add -L` now showed my key had been added, and after restarting iTerm, the repeated password prompts stopped.


<hr />

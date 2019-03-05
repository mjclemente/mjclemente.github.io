---
published: true
title: It's containers all the way down
layout: post
tags: [docker, digitalocean, containers, container orchestration]
---
I haven't had a lot of time to blog recently. Why? In a word: Docker. At work we're moving toward a fully containerized stack, from development to production, so I've needed to spend every free minute trying to learn the ins-and-outs of container orchestration. Two initial takeaways: 1) it's still not as easy as it could be, and 2) it's not as hard as some make it out to be.
<!--more-->

Don't expect to learn anything from this post; it's more of a rambling on the direction I'll likely be going with some future posts.

## Container Orchestration and Developers
One of the promises of containerization is that it will make developers' lives easier. This might be the case in larger organizations, where there are sysadmins and DevOps teams; it's likely the case in various development environments. However, I think there's still a lot of work that needs to be done in order to bring the benefits of running containers in production to small/midsized teams and businesses.

That is, the technology and tooling are moving very quickly, and established patterns and practices haven't been fully formed. The space is still, despite its rapid growth, immature. In my experience, while I hear about companies taking Kubernetes or Swarm to production, I haven't seen the corresponding blog posts, tutorials, or concrete examples explaining how solo and small-team developers could do the same.

Right now, I'm one of those small-team developers. I enjoy and focus on application development far more than DevOps, so this is a movement out of my comfort zone, for better and worse. Consequently, I'll be documenting my ongoing (hopefully successful) attempts to understand and master various aspects of our brave new containerized world.

## Learning from Docker Pros
There are some helpful resources out there (if you've found any that were particularly enlightening, please let me know). For example, if you're interested in learning Docker, I recommend [Bret Fisher's](https://www.bretfisher.com/) Udemy course, [Docker Mastery](https://www.udemy.com/docker-mastery/) and the still-under-development release of his [Swarm Mastery](https://www.udemy.com/docker-swarm-mastery/) course. A lot of what I'll be exploring will be based on Bret's demos, examples, and [GitHub repositories](https://github.com/BretFisher).

## DigitalOcean
Because the ultimate goal here is containers in production, we'll need actual remote hosts. For this, I'll be using DigitalOcean. They're affordable, developer friendly, and have a clean, intuitive interface. Docker developers that I respect, including Bret and the team at [Ortus Solutions](https://www.ortussolutions.com/) recommend them, so that's good enough for me. You can get a [$100 promo if you sign up using my referral link](https://m.do.co/c/8acbd6928587).

## Next Steps
I'm writing this, with the hope that putting it out there will actually make me follow through and do it. One of my goals is to set up and deploy my own "production" Swarm. That is, an app, codebase, and workflow that I use and can share with other developers. Future posts will, hopefully, be steps toward that goal.

___
For those interested, here's where the title of this post comes from: [Turtles all the way down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down)

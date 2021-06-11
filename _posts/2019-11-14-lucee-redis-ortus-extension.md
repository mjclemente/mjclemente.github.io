---
date: 2019-11-14
published: true
title: "Using Redis with Lucee: An Approach with the CommandBox Docker Image and Ortus Redis Extension"
layout: post
tags: [redis,lucee,extensions,commandbox,cfml,coldfusion]
---
Well, the title feels a bit like word soup, but I think it's accurate. When I got started with Lucee, containers, and external cache providers, I [blogged about using Memcached](/2018/08/17/install-lucee-extensions-on-commandbox-docker-containers.html#the-backstory-using-memcached). At work, we've since shifted our stack, and now primarily use Redis for caching. 
<!--more-->

This post outlines some reasons for that change and includes an example repo of the Lucee/Redis configuration discussed - which is to say, it is divided into two parts, [why](#why-redis) and [how](#how-to-use-redis-with-lucee), and if you're only here for the code, here's a link to the repo: [mjclemente/redis-lucee-extension](https://github.com/mjclemente/redis-lucee-extension)

A disclaimer - I'm discussing one method of containerizing a ColdFusion application with external session storage - there are *a number of ways* this can be done, and dozens considerations when determining the best approach for your particular application.

## Why Redis?

There were a handful of factors that motivated us to switch to Redis for most of our external application caches. In no particular order:

* **Insight into the cache**: While not impossible, viewing the keys/values stored in Memcached is not convenient. Redis, on the other hand, along with its [helpful CLI](https://redis.io/topics/rediscli), has a number of GUI clients - my current favorite being [TablePlus](https://tableplus.com/).[^2] The ability to easily inspect the cache during development can yield genuinely helpful insights  - like noticing that Docker healthchecks are creating thousands of rogue sessions, to use an embarrassing, real-world example. Bottom line, a lot can seem confusing and opaque when working with containers, so tools that provide insight and clarity are desirable.
* **Widespread adoption**: This is somewhat subjective, but the impression I've gotten is that Redis has "won" this space. The best concrete measurement of this that I've found is the Github and Docker Hub stars for the projects. In both cases, Redis outpaces Memcached considerably.[^1] Generally, widespread use in the developer community results in more resources for effectively configuring, using, and debugging the tool.
* **Lightweight**: Redis is a lightweight solution, with its Alpine image coming in under 10MB. This contrasts with other caching alternatives that we considered, such as MongoDB and Couchbase, whose smallest images weigh in at over 140MB and 190MB respectively. Size isn't everything - it certainly shouldn't be the deciding factor for an image or caching solution - but we did appreciate the minimal footprint when integrating Redis with our stack. 
* **Ortus Extension**: Lucee's official extensions for Redis (and Memcached) provide basic caching functionality and are free. However, both are marked BETA - their features are limited, and their ongoing development and support is an open question. Ortus Solutions, on the other hand, provides a [Redis Extension](https://www.ortussolutions.com/products/redis-lucee) which, while not free, is supported and being actively developed. It's gone through two significant releases since we started using it, supports custom namespaces for cache regions, and the Ortus team actively helped us resolve every issue we encountered while using it. Everyone's calculation on this front will be different, but some things are worth paying for.
* **Pub/Sub**: This was not a deciding factor, but we were intrigued with the Redis implementation of Pub/Sub. It's not something we're prepared to use, but it's a space that we want to explore in the future.

Our line of thought might not make sense for your applicaton or situation. But, if you're interested in taking Lucee for a spin with the Ortus Redis extension, read on.

## How to Use Redis with Lucee

Rather than simply listing and describing the steps here, I've put together an example Github repo: [mjclemente/redis-lucee-extension](https://github.com/mjclemente/redis-lucee-extension)

The instructions in the README should be sufficient for starting a Lucee container that uses Redis for the session and object cache, via the Ortus Redis Extension. If that sounds interesting, you should take it for a spin!

For those interested in the details, here's a little more information on what's going on with this particular approach. We'll start with the Dockerfile, located here: `/build/cfml/Dockerfile`

```dockerfile
# Starts from the CommandBox Docker Image
FROM ortussolutions/commandbox:4.8.0

# Declared as ENV for clarity
ENV CONFIG_DIR /config

# Copy in our build / config file(s). 
COPY ./build/cfml/config/ ${CONFIG_DIR}/
```

The config files that are copied in include the Ortus Redis Extension: `/build/cfml/config/extensions/ortus-redis-cache-*.*.*.lex`. This will be loaded and deployed when we warm up the server. 

Our application files are copied in next:

```dockerfile
COPY ./app ${APP_DIR}
```

This is important, as the `/app` folder also includes the CommandBox configuration files:
  * `box.json` - defines a dependency used to load the Ortus Redis Extension
  *  `server.json` - sets the version of Lucee
  *  `.CFConfig.json` - configures caches

Our Dockerfile then installs the `box.json` dependencies:

```dockerfile
WORKDIR $APP_DIR
RUN box install
```

The only dependency in this particular `box.json` is the `docker-lex-install` CommandBox module, which will install the extensions we copied in, later in the build process.

We've accounted for the Ortus Redis Extension, but not its license. We do that next, first ensuring that the necessary folders exist, and then moving in the trial license for the Ortus Redis Extension:

```dockerfile
WORKDIR $CONFIG_DIR
RUN mkdir -p /root/serverHome/WEB-INF/lucee-server/context/context/ortus/redis/ && \
  mv ortus.redis.license.properties /root/serverHome/WEB-INF/lucee-server/context/context/ortus/redis/license.properties && \
  touch .trial && \
  mv .trial /root/serverHome/WEB-INF/lucee-server/context/context/ortus/redis/.trial
```

We now have all the components where they need to be, and are ready to warm up the server:

```dockerfile
RUN ${BUILD_DIR}/util/warmup-server.sh
```

During this process, the server is started for the first time, using the version of Lucee defined in `server.json`. 
  * The CommandBox module `docker-lex-install` intercepts `onServerStart()`, and moves the `.lex` extension files into place for deployment/installation.
  * The `.CFConfig.json` file defines the session and object caches and the Ortus-specific settings they need to connect with Redis.

That's the end of the build process. When the Lucee/CommandBox image is actually run, the Ortus Redis Extension is installed and the caches are configured, so the "app" is ready to use, with all of its caching stored in Redis.

If you've got any questions about this approach, let me know! 

____

[^1]:In the case of Github stars, [Redis](https://github.com/antirez/redis) has 39.6K while [Memcached](https://github.com/memcached/memcached) has 9.5K. On Docker Hub, there are 7.5K stars for the [Redis Image](https://hub.docker.com/_/redis), with 1.4K for the [Memcached Image](https://hub.docker.com/_/memcached).
[^2]: You can use TablePlus for free (but there are a limitations). It also supports a wide range of relational and NoSQL databases. I've found it a fantastic tool.
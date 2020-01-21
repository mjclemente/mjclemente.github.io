---
published: true
title: "Sizing Your Redis Cache for Sessions - A Quick Lesson Learned"
layout: post
tags: [redis,lucee,extensions,commandbox,cfml,coldfusion]
---
Hindsight being 20/20 makes the lesson of this post appear comically obvious, but as the underlying issue took a while to track down, I thought it worth documenting. The embarrassing TLDR; is that you should make sure that your Redis cache is appropriately sized for your session data or you might end up with errors that are difficult to debug.
<!--more-->

I imagine the general principle discussed here applies to a wide range of ColdFusion setups in which Redis serves as an external object/session cache - whether you're using the [Lucee Redis extension](https://github.com/lucee/extension-redis), Redis as an [external cache for Adobe ColdFusion](https://helpx.adobe.com/coldfusion/using/external-session-storage.html), or, as I've [discussed before](/2019/11/14/lucee-redis-ortus-extension.html), the Ortus Redis Extension for Lucee. 

## The Error: `Connection reset	`

In our logs, we began to notice the following error:

```log
redis.clients.jedis.exceptions.JedisConnectionException: java.net.SocketException: Connection reset
```

While the error occurred regularly (a couple of times a day), we couldn't reproduce it. To make matters more frustrating, the error didn't bubble up to the ColdFusion application, so it wasn't trapped and logged by FusionReactor. Finally, we didn't directly observe the error causing any problems - nothing in the application appeared broken.

## Tracking Down the Issue

We continued to monitor the logs, eventually widening our search to other services, in an attempt to determine what was happening in the application around the time of the error. Nginx held the first clue - the error was always preceded by a user accessing the login page. This finding lead to the realization that the error only occurred during the workweek. A survey of employees confirmed that sometimes they found themselves unexpectedly logged out of the application, but that it wasn't too common, so no one brought it to our attention. 

All signs were pointing to the session cache as the culprit, but we didn't know why. Using the `redis-cli`, we eventually found that the Redis session cache was in the vicinity of the `maxmemory` we had defined. Not perilously close - there was definitely a little breathing room - but close nonetheless. That's when it clicked: **Redis was evicting valid sessions in order to safely maintain its memory limit**, which was triggering the error. Increasing the `maxmemory` for the session cache resolved the issue.

## Lesson: Sessions aren't like other caches

*Sessions are fundementally different than other types of caches* - you do not want your session data evicted because of a caching algorithm. Session data should either expire according to its TTL or be manually evicted by your application. Consequently, when setting up a Redis cache for sessions, be sure to give it *plenty* of headroom, and monitor use of the cache to ensure it's being used as expected. In our case, some of the data being stored in the session cache was substantially larger than we had anticipated.

So, that's it. Seems so obvious looking back, was it was a bit of a trip to figure it out. Lesson learned!
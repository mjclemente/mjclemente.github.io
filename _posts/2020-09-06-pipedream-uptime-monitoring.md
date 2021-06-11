---
date: 2020-09-06
published: true
title: "Building a Basic Uptime Monitor with Pipedream"
layout: post
tags: [pipedream, nodejs, serverless]
typora-root-url: ../../blog.mattclemente.com
---
During a [recent live-coding session](https://www.youtube.com/watch?v=Ce7nvF45GNk), I tried to build a website uptime monitor with [Pipedream](https://pipedream.com/). Even with a few digressions, I managed to get most of it done within the hour, and figured that the process and platform were worth sharing.
<!--more-->

## Some Pipedream Context

If you're unfamiliar, Pipedream is a severless platform with a particular focus on making it easy for developers to integrate with external services. I was introduced to it as "Zapier + AWS Lambda," which doesn't quite do it justice, but does convey the general idea.[^1]

Pipedream provides the ability to build [workflows](https://docs.pipedream.com/workflows/), which are a series of steps that can run code (Node.js) and interact with external services. These can be [triggered in a variety of ways](https://docs.pipedream.com/workflows/steps/triggers/#app-based-triggers), including email, HTTP/webhooks, a growing number of apps, and of particular interest to this post, via a cron schedule[^2]. Within the Node.js steps, you can install and use most npm packages, and you can set and pass variables along the steps of your workflow.

All of this seemed pretty powerful, and after getting a feel for the platform, I wanted to use it to make something practical. So, I set out to make a workflow for monitoring my blog's uptime.

## The Uptime Monitor Workflow

So as not to bury the lede too far, here's the [finished workflow](https://pipedream.com/@mjclemente/website-monitor-p_KwCrmN/readme), that you can use or customize to suit your needs. And here's an outline of the steps it currently uses:

1. **Trigger** - Because I wanted to check the uptime of my blog regularly, I used the [Cron Scheduler](https://docs.pipedream.com/workflows/steps/triggers/#cron-scheduler). Depending on how aggressive you want to be with monitoring, you could have it run the workflow every minute. Just keep in mind that these will eat into your daily free compute limit[^3], so a more relaxed schedule - say every 5 minutes - might be more appropriate for a non-critcal website.
2. **Make the HTTP request** - I'm using [axios](https://www.npmjs.com/package/axios), but you could use [any number of approaches](https://www.twilio.com/blog/2017/08/http-requests-in-node-js.html) to making the HTTP request. When building your workflow, you'll want to make sure that it's handling error response codes properly. I used [httpstat.us](https://httpstat.us/) to simulate 400 error codes while testing. I also decided that monitoring the response time would be helpful, so I added this, using [axios request/response interceptors](https://github.com/axios/axios#interceptors).
3. **Determine if the website is down** - Based on the result of the HTTP request, this step returns a boolean parameter, indicating if the website is down. Nothing fancy here - if it doesn't return a 200 response, it's considered down.
4. **Send an alert email (or not)** - The job of this step is to send the downtime alert email. However, in most cases the website isn't down. There isn't a built-in way to skip steps in a workflow, so the body of this function is wrapped in an if-statement, evaluating the boolean result of the previous step. If it's not down, it doesn't send the email.
5. **Bonus - track metrics with Datawaves** - While not necessary for uptime monitoring, this additional step provides some interesting reporting/analytics about the website being monitored. While browsing Pipedream's [list of integrated apps](https://docs.pipedream.com/apps/all-apps/#apps), I came across [Datawaves](https://datawaves.io/) - an event analytics platform that provides a generous free tier. So, I set up a free account and the results of the uptime check (status code, status text, and request duration) get pushed into Datawaves. In their dashboard, I can easily analyze the results of my uptime checks - average response duration, number of downtime results, etc. Pretty nifty!

And that's it - a free, and pretty powerful website uptime monitor with Pipedream.

Now, there's certainly more that we could do to improve this workflow. For example, we could limit the frequency with which the error alert emails are sent - because if you're checking the uptime every minute, you probably don't want an email every minute about it being down. We could also build in some degree of fault tolerance; for example, if you're checking every minute, don't send the alert unless you get two consecutive downtime readings. And, while we're brainstorming, we could abstract the workflow further, triggering it via a webhook and passing in the website dynamically; that way we could use it to monitor multiple websites. All that said, I think it's a perfectly sufficient starting point.

Finally, if you're interested in trial and error, here's the live-coding session where I started the project.

<div class='embed-container'>
  <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Ce7nvF45GNk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>
_____

[^1]: Looking for more? This post, [*Introducing Pipedream*](https://medium.com/@todsacerdoti/introducing-pipedream-bbca9dde0dc6), by one of the founders, provides a bit more clarity and depth, along with the opening section of their documentation: https://docs.pipedream.com/
[^2]: I was today years old when I learned that *cron* was derived from *cronos*, Greek for time. Go, go [Wikipedia](https://en.wikipedia.org/wiki/Cron).
[^3]: At the time of writing, this was [30 minutes (1,800,000 milliseconds) per day](https://docs.pipedream.com/limits/#execution-time-per-day) for your Pipedream account.

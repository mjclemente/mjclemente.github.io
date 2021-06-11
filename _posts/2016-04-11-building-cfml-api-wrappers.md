---
date: 2016-04-11
published: true
title: CFML API Wrappers
layout: post
tags: [coldfusion,cfml,salesforceiqcfc,screenshotlayercfc,api]
---
There are two small projects I've been working on: [salesforceiqcfc](https://github.com/mjclemente/salesforceiqcfc) and [screenshotlayercfc](https://github.com/mjclemente/screenshotlayercfc). They are basic, CFML API wrappers for the [SalesforceIQ](https://api.salesforceiq.com/#/curl) and [Screenshotlayer.com](https://screenshotlayer.com/documentation) APIs, respectively. Their benefit to me is twofold: 1) I can use the actual functionality of the APIs in my applications, and 2), the exercise of writing the wrappers got me to think about APIs, my code, and open source code in a much more engaged way.<!--more-->

These were the first API wrappers I've written, as well as my first real open source projects. While I don't think there's a very large audience for them, hopefully someone finds them useful. I'd love feedback if anyone else ends up using them.

While each API had its quirks and issues, overall, writing the wrappers was fairly straightforward, thanks to working off of the framework that John Berquist ([jcberquist](https://github.com/jcberquist)) built out with [stripecfc](https://github.com/jcberquist/stripecfc). It's really well designed, providing clear, concise public methods that pass the necessary arguments on to the private interactions with the API. This structure makes adding new methods easy to implement (once the quirks of the specific API have been accounted for).

Working on these API wrappers really highlighted the need for writing actual unit tests. Not using them (unit tests) is a personal deficiency I've been vaguely aware of for quite some time. However, manually testing each method over and over as I worked on the API made the value of a testing framework painfully clear. I know that I should be using [TestBox](https://www.gitbook.com/book/ortus/testbox-documentation/details), and I'm hoping to get started with it soon. I've started working on a CFML wrapper for the [MailGun API](https://documentation.mailgun.com/api_reference.html), using the same framework, but given that it's much larger, I don't know when that will be done. It would probably be a good place to begin unit testing as well. We'll see.
<hr />
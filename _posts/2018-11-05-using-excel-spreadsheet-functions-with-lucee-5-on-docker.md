---
published: true
title: How Not To Use Spreadsheet Functions in Lucee
layout: post
tags: [lucee, extensions, docker, coldfusion, cfml, spreadsheets, excel]
---
It's not difficult to use ColdFusion spreadsheet functions (e.g. `spreadsheetNew()` and `cfspreadsheet`) in Lucee. Nevertheless, I managed to make a handful of mistakes while implementing them. I've catalogued my missteps here, along with the approach that actually worked.
<!--more-->

Adobe ColdFusion provides a number of [spreadsheet functions](https://helpx.adobe.com/coldfusion/cfml-reference/coldfusion-functions/functions-by-category/spreadsheet-functions.html). There are a range of reasons you might want these in Lucee, from convenience to necessity. In my case, an application being migrated from a Adobe ColdFusion (conventional installation) to Lucee (on Docker) made widespread use of spreadsheet functions.

### Mistake #1: Assuming Spreadsheet Functions Are Supported

You'd be forgiven if, like me, you assumed that Lucee supported spreadsheet functions. After all, a few of these functions are listed in Lucee's official documentation.[^1] Admittedly, the documentation is sparse - but it's there.

![Documentation for Lucee SpreadSheetWrite][lucee-spreadsheetwrite]

There's also no indication on CFDocs.org that Lucee/Railo do not support spreadsheet functions.[^2]

I'm here to tell you that **Lucee does not support spreadsheet functions out of the box**. However, here's the good news: there are Lucee extensions to add this functionality.

### Mistake #2: Using the Wrong CFSpreadsheet Extension

Adobe's approach with ColdFusion is to bake in all the functionality they can - Lucee takes a more modular approach, providing specialized functionality, like spreadsheet manipulation, via extensions.

After a few searches, I reached the conclusion that the community had settled on [Leftbower/cfspreadsheet-lucee](https://github.com/Leftbower/cfspreadsheet-lucee) as the preferred extension for replicating Adobe's spreadsheet functionality... *except that I didn't quite read the docs closely enough*. If I had slowed down, just a little, I would have seen that there's an updated version of the extension for Lucee 5: [Leftbower/cfspreadsheet-lucee-5](https://github.com/Leftbower/cfspreadsheet-lucee-5).

Bottom line, there are two versions of the extension - use the one that matches your Lucee install. For us, this was Lucee 5.

It's worth noting, at this point, Julian Halliwell's [cfsimplicity/lucee-spreadsheet](https://github.com/cfsimplicity/lucee-spreadsheet) library. It's a commonly used approach for handling spreadsheets in Lucee and it looks like it provides dozens of very helpful [functions not found in ColdFusion](https://github.com/cfsimplicity/lucee-spreadsheet#extra-functions-not-available-in-coldfusion). However, be aware that it does not replicate Adobe's syntax - using the library requires rewriting the portions of your application that interact with spreadsheets. So, while it's an excellent solution for some use cases, we didn't want to take this approach while migrating our app.

### Mistake #3: Using `box install`

The extension we settled on, *cfspreadsheet-lucee-5*, is not an [official Lucee extension](https://download.lucee.org/#ext), but it is [listed on ForgeBox.io](https://forgebox.io/view/037A27FF-0B80-4CBA-B954BEBD790B460E), which [became a Lucee Extension Provider](https://www.ortussolutions.com/blog/new-forgebox-features-march-2018) in March of 2018.[^3] On the ForgeBox listing for the module, there's an install command: `box install 037A27FF-0B80-4CBA-B954BEBD790B460E`

![ForgeBox Install command for cfspreadsheet][forgebox-install-command]

**This installation command does not work**, nor is it supposed to work. The boilerplate text here is only applicable to the other types of modules listed on ForgeBox; Lucee extensions can't be installed with CommandBox. If you find this confusing, vote for the [support ticket to correct the installation instructions](https://ortussolutions.atlassian.net/browse/FORGEBOX-162).

### The Very Simple Approach that Worked

So, if you want to use ColdFusion spreadsheet functions in Lucee:

* **For conventional server installations**: Add ForgeBox.io as an extension provider in the Admin and then install the *cfspreadsheet-lucee-5* extension.
* **For Docker deployments**: Take a look at my [previous post about approaches to installing extensions](/2018/08/17/install-lucee-extensions-on-commandbox-docker-containers.html). I used the `docker-lex-install` CommandBox module discussed in that post to import the [cfspreadsheet-lucee-5.lex](https://github.com/Leftbower/cfspreadsheet-lucee-5/blob/master/cfspreadsheet-lucee-5.lex) file during my server warmup.

Happy spreadsheeting!

___
[^1]:For example, [`SpreadSheetWrite`](https://docs.lucee.org/reference/functions/spreadsheetwrite.html) and [`SpreadSheetNew`](https://docs.lucee.org/reference/functions/spreadsheetnew.html). If I get around to it I'll put in a PR to have these confusion pages removed.
[^2]: Again, [`SpreadSheetWrite`](https://cfdocs.org/spreadsheetwrite) and [`SpreadSheetNew`](https://cfdocs.org/spreadsheetnew) can serve as examples of this, along with [others](https://cfdocs.org/spreadsheet%2Dfunctions).
[^3]: There is a support ticket to [add ForgeBox as an extension provider, out of the box](https://luceeserver.atlassian.net/browse/LDEV-1735). Go forth and vote for it, if you'd like that!

[lucee-spreadsheetwrite]: /public/assets/images/lucee-spreadsheetwrite.png
[forgebox-install-command]: /public/assets/images/cfspreadsheet-forgebox-install-command.png
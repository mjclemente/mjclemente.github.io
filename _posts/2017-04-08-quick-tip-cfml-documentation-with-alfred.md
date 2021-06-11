---
date: 2017-04-08
published: true
title: Quick Tip - CFML Documentation on Mac
layout: post
tags: [coldfusion,cfml,documentation]
---
I'm frequently digging up documentation for various CFML functions. The key, for me, is the convenience and ease with which I can integrate these lookups into my workflow. There are a few ways to do this, depending on your preferences, but I'm happy with the process I've settled on, as it provides fast access to both [Adobe's ColdFusion documentation](https://helpx.adobe.com/coldfusion/home.html) and [cfdocs.org](http://cfdocs.org/).

<!--more-->

## Adobe Documentation

[Adobe's ColdFusion documentation](https://helpx.adobe.com/coldfusion/home.html) has improved, of late; however, I find navigating the website difficult. Rather than accessing it directly, I find using the offline docset viewer [Dash for Mac](https://kapeli.com/dash) far more convenient and direct.[^1]. Check a few boxes in Dash, and you're good to go:

![Select docsets in Dash](/public/assets/images/dash-app-select-docsets.png)

Accessing the docs can be streamlined even further by incorporating [Alfred](https://www.alfredapp.com/). In Dash, add the Alfred integration:

![Alfred Dash integration](/public/assets/images/alfred-dash-integration.png)

And now you can quickly search the docs for functions by prefixing your Alfred search with 'cf': 

![Searching ColdFusion documentation in Dash with Alfred](/public/assets/images/searching-coldfusion-dash-documentation-with-alfred.png)

![CFFunction Dash documentation](/public/assets/images/cffunction-dash-documentation.png)

## CFDocs.org

I've found myself shifting from Adobe's documentation to using the community driven [cfdocs.org](http://cfdocs.org/). There are a few reasons for this: 1) The script documentation is more extensive, 2) there are more examples, 3) and the examples helpfully link to [trycf.com](http://trycf.com/), so I can run them right in the browser, if necessary. The easy-to-use URL format (cfdocs.org/ `function-or-tag-name`) makes it simple to hit the tag or function I'm looking for. 

I was just hoping for a slight more streamlined workflow, more akin to the Dash process that I use for the official Adobe documentation. As Pete Freitag explained, [you can add a custom search in Chrome for cfdocs.org](https://www.petefreitag.com/item/838.cfm). This was good, but it's actually possible to add the search directly to Alfred, basically replicating the Dash workflow.

In the Alfred preferences, under the "Web Search" tab, you can "Add a Custom Search". So, I added the new search URL: `http://cfdocs.org/{query}`, titled it *CFDocs*, and set the keyword to `cfd`.

![Add a custom cfdocs.org search to Alfred](/public/assets/images/add-custom-cfdocs-search-to-alfred.png)

Once I saved that, I basically had a duplicate workflow for accessing the [cfdocs.org](http://cfdocs.org/) docset; Alfred keyword -> documentation, from wherever I'm working.

![CFDocs.org search in Alfred for cffunction](/public/assets/images/cfdocs-search-in-alfred-for-cffunction.png)

![CFDocs.org for cffunction](/public/assets/images/cfdocs-for-cffunction.png)

I've been happy with the result; using the combination of Alfred and Dash / cfdocs.org, I've got fast access directly to the documentation I need, whenever I need it.

<hr>
[^1]: Unsurprisingly, it provides far more than just ColdFusion documentation; there were over 150 docsets the last time I checked; and they're kept up to date. I believe it's also possible to manually add the Lucee documentation. The app is paid, and whether you think it's worth it probably comes down to your general attitude towards paying for software. 
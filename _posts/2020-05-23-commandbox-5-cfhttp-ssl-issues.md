---
published: true
title: "CFHTTP and SSL issues with CommandBox 5"
layout: post
tags: [commandbox,cfml,coldfusion,cfhttp,ssl]

---

We recently ran into a puzzling issue with `cfhttp` and CommandBox 5. Requests to certain domains, such as [trycf.com](https://trycf.com/), would fail with `Unknown host: Received fatal alert: handshake_failure`, despite the domain's SSL certificate being valid. I hope this post, which digs into the somewhat unexpected source of them problem, is helpful to anyone else who might encounter the error.
<!--more-->

**tldr;** Try adding the Runwar arg `--ssl-eccdisable=false` to your `server.json` file to resolve `cfhttp` connection issues.

* TOC
{:toc}
## The Issue

While most `cfhttp` requests worked as expected, we found our application unable to make requests to certain domains - a problem first, because one of the domains was a geocoding API we used, and second, because we didn't know why it was happening.

We took detours down a few rabbit holes, trying different versions of Java and tweaking a host of JVM args, but nothing worked; everything came back with an SSL handshake error.

Finally, we discovered that when our app had SSL enabled in `server.json`, the `cfhttp` requests would fail. If we disabled SSL, the requests would be completed successfully.

This, of course, made no sense and was the last place we thought to look. I put together a [repro](https://github.com/mjclemente/commandbox-ssl-cfhttp-repro) and opened a ticket for the issue: [CommandBox-1173](https://ortussolutions.atlassian.net/browse/COMMANDBOX-1173).

## The Runwar Argument

Brad Wood, wizard of CommandBox that he is, responded to the ticket, asking me to try adding the following Runwar argument: `--ssl-eccdisable=false`. Sure enough, with that flag added, the formerly failing `cfhttp` requests worked, both with and without SSL enabled. 

  Here's what the flag looks like in a `server.json` file:

```json
{
  "runwar":{
      "args":"--ssl-eccdisable=false"
  }
}
```

However, there's a bit more to the equation, because the way your application responds to this argument depends on the ColdFusion engine (Adobe, Lucee), the particular version, and the version of Java being used. And the results vary considerably -  more on this at the end.

### Why the Runwar Argument is Needed

It turns out that earlier versions of Adobe ColdFusion, running via CommandBox, had issues serving SSL certificates,[^1] unless they were started with the JVM arg:

```
-Dcom.sun.net.ssl.enableECC=false
```

ECC stands for Elliptic Curve Cryptography, but I'm not going to pretend that I understand it any more than that.

The bottom line is that by disabling this ECC option, ColdFusion/CommandBox could serve SSL certificates without issue. Consequently, the flag disabling ECC got [baked into CommandBox as a default](https://github.com/Ortus-Solutions/runwar/blob/411d91167d7cfed6ee450f497fa54e464cc2e5a5/src/main/java/runwar/options/ServerOptionsImpl.java#L121) and is [applied when starting a server](https://github.com/Ortus-Solutions/runwar/blob/411d91167d7cfed6ee450f497fa54e464cc2e5a5/src/main/java/runwar/Server.java#L264) with SSL enabled. In an earlier version of Runwar that was used with CommandBox 4.8, this flag was only applied to Adobe ColdFusion servers, but because of how things were refactored, it now appears to be used across the board.

None of this mattered all that much, because there were no known negative side effects of passing in this option.

### Considerations and Negative Side Effects

It turns out that enabling/disabling the ECC flag impacts not just how your application serves SSL, but also how it interacts with SSL certificates on other websites. For some current SSL certificates[^2], disabling ECC renders ColdFusion unable to negotiate an SSL connection when making a `cfhttp` request. The flag Brad provided overrides the CommandBox default, so the ECC option remains enabled, and ColdFusion can complete the `cfhttp` request successfully.

Unfortunately, you can't just blindly implement `--ssl-eccdisable=false`. If you have SSL enabled and are running Adobe ColdFusion 11, this option will prevent your app from loading because of an SSL protocol error. The same holds true for Adobe ColdFusion 2016, if you're running Java 8, though the protocol error is resolved if you're running Java 11.

That said, if you have SSL enabled and you're running any version of Lucee 5 or Adobe ColdFusion 2018, based on my limited testing, you'll likely want to configure your servers with `--ssl-eccdisable=false`, to ensure the widest range of successful `cfhttp` interactions with SSL certificates.

## Compatibility Matrix

The table here should the results of my testing. The **Runwar Arg** column indicates if the server was started with the `--ssl-eccdisable=false`  flag. If the SSL connection was valid, I made `cfhttp` requests to both [https://trycf.com](https://trycf.com) and [https://google.com](https://google.com). Hopefully you find this helpful.

That said, there is no substitute for testing your own apps.

<input type="search" class="light-table-filter" data-table="order-table" placeholder="Filter">
<table class="order-table table">
  {% for row in site.data.commandbox-cfhttp-ssl-matrix %}
    {% if forloop.first %}
    <thead>
    <tr>
      {% for pair in row %}
        <th>{{ pair[0] }}</th>
      {% endfor %}
    </tr>
    </thead>
    {% endif %}

    {% tablerow pair in row %}
      {{ pair[1] }}
    {% endtablerow %}
  {% endfor %}
</table>
<script src='/public/assets/js/light-js-table-filter.js'></script>

**Specific versions used for testing**

* Adobe Coldfusion 2018.0.9+318650
* Adobe ColdFusion 2016.0.15+318650
* Adobe ColdFusion 11.0.19+314546
* Lucee 5.3.6+61
* Lucee 5.2.9+31
* Java (OpenJDK) 11_jdk-11.0.6+10
* Java (OpenJDK) 8_jdk8u252-b09.1

____

[^1]: For an example of the errors/issue, see [this discussion in the CommandBox forums](https://groups.google.com/a/ortussolutions.com/forum/#!topic/commandbox/zCNk43F3P94/discussion).
[^2]: One commonality in the websites that we were unable to connect to was that their certificates were issued by Cloudflare. For example: [https://trycf.com](https://trycf.com), [https://geocoder.ca](https://geocoder.ca), and [https://jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com)
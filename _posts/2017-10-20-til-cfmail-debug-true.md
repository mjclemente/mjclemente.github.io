---
published: true
title: TIL - cfmail debug=true
layout: post
tags: [coldfusion,til]
---
I'm speaking next month at [Adobe ColdFusion Summit 2017](https://cfsummit.adobeevents.com/) in Las Vegas. In preparation, I was reading the documentation for [`cfmail`](https://cfdocs.org/cfmail) and was surprised to learn that it had a boolean `debug` attribute. While it might not be a hidden gem, if you're sending emails via STMP, you might find this option helpful to, yes, *debug* email issues.
<!--more-->

Adobe's documentation for [cfmail](https://helpx.adobe.com/coldfusion/cfml-reference/coldfusion-tags/tags-m-o/cfmail.html) has a fairly straightforward explanation of the attribute; it's an optional boolean that defaults to `false`. When set to `true` it

> sends debugging output to standard output. By default, if the console window is unavailable, ColdFusion sends output to cf_root\runtime\logs\coldfusion-out.log on server configurations.

While interesting, this left me with some big questions. Are these logs viewable through the Administrator? And what exactly gets logged?

## Locating the Log Files
I ran some initial tests and saw nothing in the Administrator. I also couldn't find the log file referenced, on my CommandBox instance or on our Dev server. Just when I thought that perhaps I needed to file a bug report, I located the files. On the server, they were in

`cf_root\logs\coldfusion-out.log`

In CommandBox instances, I found them here: 

`~/.CommandBox/server/{Server}/{CF Engine}/logs/server.out.txt`

## STMP Debugging Output

So what actually gets sent to the log file? Pretty much everything; it appears to be a full SMTP conversion, complete with the full, raw content of the email. Take a look:

![cfmail smtp debugging conversation](/public/assets/images/cfmail-smtp-debugging-conversation.png)

If you were needed to check on message headers, rendered content, or sort out some other email issues, I imagine this would be immensely helpful.

## Tailing the Log

If you're a CFML developer who is intimidated by the phrase 'tailing a log file'... don't be.[^1] It's a simple process for reading, in real-time, the contents of a log file. I prefer this approach, rather than needing to log into the ColdFusion Administrator and refresh the Log Files page, waiting for a change.

Tailing the log files mentioned above, while the `cfmail` debugging attribute is set to true, provides a live stream of the emails your application is sending. Most people are familiar with tailing on Linux/Mac: 

```shell_session
$ tail -f ~/path/to/the/log/file.log
```

You can also do this in Windows now, using Powershell. For example:

```shell_session
$ Get-Content  cf_root\logs\coldfusion-out.log -Tail 2 -Wait
```

Two final notes; keep in mind that  1) the log files mentioned include a lot more information than just `cfmail` debugging, and 2) debugging `cfmail` generates a lot of output, so keep an eye on the size of your log files. Happy emailing!
<hr>
[^1]: Nando Breiter has a helpful [introductory blog post](http://dnando.github.io/blog/2015/06/16/debugging-technique-cfml-development/) on the topic.


  

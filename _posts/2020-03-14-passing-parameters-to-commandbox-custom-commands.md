---
published: true
title: "Passing Parameters to CommandBox Custom Commands"
layout: post
tags: [commandbox,cfml,coldfusion]

---

Building on my previous post, which covered writing a basic custom command for the CommandBox CLI, I put together another [video](https://www.youtube.com/watch?v=xpmQ918Di_c), detailing how to pass parameters to your command.
<!--more-->

Odds are, if you're interested in writing a custom command, you're going to need yours to handle parameters. CommandBox provides a number of parameter-specific conventions and features, which serve the dual-purpose of making commands both easier to write and use. In the video I cover:

- Handling named and positional parameters, as well as boolean flags
- Providing hints/help for parameters
- Adding static and dynamic tab-completion
- Handling file-system parameters

The [documentation on using parameters](https://commandbox.ortusbooks.com/developing-for-commandbox/commands/using-parameters) is thorough, clear, and covers the subject in a bit more detail than I do in this video. The docs are a great resource if you've got questions - I'm constantly going back to them, and more often than not, I find that CommandBox already provides an elegant approach to whatever I'm trying to accomplish. Best of luck writing your own custom commands!

<div class='embed-container'>
  <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/xpmQ918Di_c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

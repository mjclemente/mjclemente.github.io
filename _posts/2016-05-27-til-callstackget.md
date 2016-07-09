---
published: true
title: TIL - callStackGet()
layout: post
tags: [coldfusion,til]
---
This wasn't supposed to be long post - just a quick write-up on a helpful function that I stumbled upon. It never ceases to amaze me when I encounter an aspect of ColdFusion (function, feature, tool, etc) that's new to me, but has been around for years. This time, the function is `callStackGet()`, which was apparently introduced in ColdFusion 10, but I had never encountered before. 
<!--more-->

The backstory, in this case, is that I was putting together a small logging service to use in a FW/1 app. This wasn't for site-wide error handling - just keeping track of potential problem areas, like file uploads, usually within try/catch blocks. I wanted it to provide as much helpful information as possible, in a consistent manner, while keeping the code footprint small. Basically, I wanted to be able to log a small message, and then have all relevant information included automatically.

Given that it was a FW/1 app, I knew I could easily include the top level CFC and function called, via `getSubsystemSectionAndItem()`. I also considered using `GetFunctionCalledName()`, which, depending on how I incorporated it, would be able to give me the actual function where the logging occurred. However, I was hoping to get a little more, specifically, the template and line number.

Of course, when I searched Google for some variation on "ColdFusion get line number", the top result was a [blog post by Ben Nadel](http://www.bennadel.com/blog/2346-coldfusion-10---accessing-the-call-stack-with-callstackget.htm). This was the first I had heard of the awkwardly named `callStackGet()`, which, as the docs state:

> Returns an array of structs. Each struct contains template name, line number, and function name (if applicable). This is a snapshot of all function calls or invocations.

This is really neat - it's a window into exactly what's happening in your application, and the order in which it happens. Here's a screenshot of what the result can look like:

![callstackget example](/public/assets/images/callstackget-example-array.png)

The first item in the array is the most recently executed function, followed by previous function calls, all the way back to the originator. There's probably a lot that can be done with this - Ben's post is about using it, somehow, in UDFs. For my purposes, however, it was a simple mechanism for providing helpful information about logging context. If the function is new to you too, I encourage you to check it out: [http://cfdocs.org/callstackget](http://cfdocs.org/callstackget).

## The Resulting Logging Service (if you're interested)

In the end, the log service provided a (very) simple wrapper for the `writeLog()` function. The text that gets logged is built by taking the basic developer-supplied message and augmenting it with information about the environment in which the code was executed. So, the logging call looks something like this:

	variables.logUtility.log( text = 'File Delete Failed', method = GetFunctionCalledName() );
	
The "method" argument is optional - it's a simple way to grab the actual name of the function that was called. Not necessary, but convenient. The log function itself takes the text, and uses the call stack to add the template and line number in question before writing the log.

	public void function log( string text, string logFile = "application", string type = "Error", boolean displayApplication = true, string method = "" ) {

	    var message = text & ' - ' & 'Template: #getLogTemplateFromCallStack().template# - Line Number: #getLogTemplateFromCallStack().lineNumber# - FW/1: #variables.fw.getSubsystemSectionAndItem()# - Method: #method#';

	    writeLog( text="#message#", type="#type#", application="#displayApplication#", file="#logFile#" );
	  }

You'll see `log()` has additional optional arguments that basically map onto `writeLog()`. The function that actually uses `callStackGet()` looks like this:

	private struct function getLogTemplateFromCallStack() {
		return callStackGet()[3];
	}

Because it is executing `callStackGet()`, and it is being called by the `log()` function, it needs to reach back 3 positions in the array to get information about the template/function being logged. The final result is a clean, helpful, consistently formatted log message that looks like this:
	
```text
File Move Failed - Template: D:\webroot\applicationname\model\services\file.cfc - Line Number: 84 - FW/1: applicationname:files.savefile - Method: uploadFile 
```
	
	
It's easy to read, provides the necessary information, and makes it really simple to begin addressing the potential problem. 
---
date: 2016-03-01
published: true
title: On Wrapping (cf)http Calls in CFThread
layout: post
tags: [coldfusion,cfml,cfthread]
---
We recently migrated to [Mailgun](http://www.mailgun.com/) (from [SES on AWS](https://aws.amazon.com/ses/)) and in general have been thrilled with the change. The API is straightforward, testing is built in, and the data available is much, much better. It's been a joy to use. We did, however, run into one issue when we made the switch. <!--more-->

While using Amazon SES, we had been sending emails via SMTP with `<cfmail>` and it went off without a hitch[^1]. When we switched to using MailGun's API[^2], we found that, on occasion, we'd get a really long response time. After a few user complaints, we used [Fusion-Reactor](http://www.fusion-reactor.com/) to track down the issue, which actually wasn't limited to MailGun; the GetResponse API, for example, was another that would occasionally lag. The slow response times were not frequent, but on occasion they reached 30 seconds (vs the more typical 200ish ms).

An [older blog post from Seb Duggan](http://sebduggan.com/blog/using-cfthread-to-speed-up-your-web-service-calls/) pointed me in the direction of using CFThread to address the issue. As he points out, for a large portion of API calls like this, particularly with transactional emails, your application does not need to know the result; you, as the developer, just need to know if it fails. 

So, the obvious solution is to move the MailGun API calls into a separate thread so the regular page processing can continue uninterrupted.

I'll be honest, I haven't used CFThread that much; I never bothered digging into thread scopes and error handling. I read lines in the documentation like: "*Each thread has three special scopes*" and "*You cannot use page- or application-based error handling techniques to manage errors that occur during thread execution*" and figured it just wasn't worth the hassle. Except, in this case, it definitely was.

Seb's post discusses handling this with tags. Here's the cfscript based setup that we implemented:

	thread
	  	name = "email_#createUUID()#"
	  	attributeCollection = structAvailableAsThreadAttributes,
	  	action = "run"
	  	priority="LOW" {

	    try {

	      	//API INTERACTION HERE
	      	//items passed into the thread attributeCollection are available here, as "attribute.variableName"

	    } catch (any e) {

	      	writeLog( text="Type: #e.type# - Detail: #e.detail# - To: #attributes.eml# - Subject: #attributes.subject#", application="yes", file="cfthread", type="Error");

	      	savecontent variable="errorDump" {
	        	writeDump( var = e, format = 'text', label = 'CFCATCH Dump' );
	        	writeDump( var = attributes, format = 'text', label = 'Attributes Dump' );
	        	writeDump( var = variables, format = 'text', label = 'Variables Dump' );
	      	}

	      	//Perhaps email the errorDump or a write to a file or some other handling
	    }
	}

With this setup, users are not impacted by the API call, and we're alerted if any errors are encountered (and, even better, they don't experience the error). Thread scopes were not that complicated, and the `try/catch/log` setup ensured that we wouldn't miss errors. Problem solved, application upgraded, and a little more knowledge acquired.
<hr>

[^1]: Other than Amazon's complete lack of tracking, analytics, reporting, logs, supression management, or any of the other features that make MailGun awesome. Seriously, you should use it.
[^2]: I'll do another post on interacting with their API. An easy enough place to start is Dominic Watson's [cfmailgun](https://github.com/DominicWatson/cfmailgun). We build our own component to interact with it, but that's a post for another day.





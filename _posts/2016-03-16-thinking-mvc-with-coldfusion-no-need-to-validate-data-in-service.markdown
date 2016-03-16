---
published: true
title: Think MVC&#58; On Validating in Services
layout: post
tags: [coldfusion,cfml,fw/1,mvc]
---
I wrote procedural CFML for a long time. Even though I've moved on to using [FW/1](http://framework-one.github.io/), I'm still trying to develop the habit of "thinking MVC". Every once in a while I have an "ah ha" moment, where I realize that I've been thinking procedurally, and there's a better way to structure my code. This is a small instance of my gradual shift from a procedural to MVC mindset.  <!--more-->

The reduction of repetitive code is one of my favorite benefits of using an MVC framework. Where I used to have nearly identical queries littered across my `.cfm` and `.cfc` templates, a service (no pun intended) now serves as the single point of contact with the database for any particular type of data my application uses. My realization today was another occasion where, but approaching my application with an MVC mindset, I was able to remove unnecessary code.  

Within each service, typically I have a `get( id )` function, which used an object's identifier to run a query and retrieve it from the database. I've always wrapped the actual query in a conditional statement, to make sure that the `id` passed in is valid:

	if (len(id) && isNumeric(id)) {
		//query
		//populate bean with result
	}

As I was doing this for the hundredth time, I stopped and asked myself why I was including this check. My thinking went something like this: 1) The query is not directly exposed to the user anywhere 2) I'm the only one calling it and passing in the `id` argument 3) Therefore, if, for some reason, the `id` isn't a valid number, I want this code to error, so I can fix the place where I passed in the invalid data in the first place.

Removing the conditional statement and running the query without validating the `id` seemed like the proper approach. The resulting code would be tighter and more purposeful. The more I thought about it, the more the idea of including validation in my service seemed wrong. However, not trusting my own judgment, I hopped on the [CFML Slack channel](http://cfml-slack.herokuapp.com/) for some confirmation. [Sean Corfield](http://seancorfield.github.io/) was gracious enough to respond, confirming my approach, and including these bits of insight:

> I generally let low-level code fail with an exception for bad values.

> My controllers validate and format data from users. My services rarely do any validation (since they should never receive raw user data).

> I only validate if continuing with a bad value would give the *wrong* results instead of just an exception (i.e., you couldn’t easily tell the difference from the outside).

> Note that I’m also happy to return nothing when there are *no results* and use `isNull( result )` in the calling code to detect that — if it matters.

Sean, along with being very generous with his time, has a deeply reasoned approach to coding. He knows and can clearly articulate *why* he has made any particular design choice. I found his responses very helpful in deepening and clarifying my own understanding of how best to structure an MVC application. 

The end result, for me, was less code in my services, and a confirmation that I'm gradually improving my approach to building applications.





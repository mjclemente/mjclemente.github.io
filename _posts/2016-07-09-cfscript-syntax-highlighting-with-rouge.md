---
published: true
title: CFScript Syntax Highlighting with Rouge!
layout: post
tags: [coldfusion,cfml,rouge,jekyll]
---
This is just an update on the cfscript lexer for Rouge. Here's the TLDR:  **It's available!** Want to get started? Read on.
<!--more-->

My [PR](https://github.com/jneen/rouge/pull/492) was merged (back on June 14) by the incredibly gifted [@jneen](https://github.com/jneen). So *you can highlight your cfscript code snippets* in Rouge v1.11.1+. If I seem excited, it's because I am - it's been a fairly long process. I waited on posting, because the [github-pages gem](https://github.com/github/pages-gem) hadn't updated its Rouge dependency yet, so I couldn't actually demonstrate the highlighting, until now...

If you have [Jekyll set up the same way that I do](/2016/02/24/getting-started-with-jekyll-part-3.html), with Bundler, then the first thing (really the only thing) you need to do to get this set up locally is update your gem dependencies:

```shell_session
$ bundle exec gem update github-pages
``` 
That will get you the version of Rouge necessary to write your cfscript code blocks like this:

~~~text
```cfc
private struct function iAmAFunction() {
	var something = {};
	return something;
}
```
~~~
The `cfc` at the start of the code block indicates the intended syntax highlighting, which in turn, should render this:

```cfc
private struct function iAmAFunction() {
	var something = {};
	return something;
}
```
And that's it. Blogging with cfscript just got a lot more colorful! 

<hr />
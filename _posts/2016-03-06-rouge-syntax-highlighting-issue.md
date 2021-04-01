---
published: true
title: Rouge Syntax Highlighting Issue
layout: post
tags: [jekyll,rouge,markdown]
---
One of the most regularly occuring frustrations in development is spending time working through what seems like a complex problem, only to discover, in the end, that it was caused by something embarrassingly simple. For better or worse, sometimes that's just the way it is -  the answer isn't obvious until you've reached it the long way around, especially early in the process of learning something new. That's what happened to me here, so this post likely won't be very instructive; just another log of me taking the long way around to reach a simple answer. <!--more-->

The backstory: After publishing my last post, I saw that when it was served from Github Pages, the code highlighting was off wherever I had used curly braces in inline code.

![syntax highlighting issue](/public/assets/images/rouge-syntax-highlighting-error.png)

When I had viewed it locally, before publishing, it displayed normally, without the garish background color and there were no errors or warnings in my local build process. Being new to kramdown, Jekyll, Liquid, Rouge, Pygments, and Github Pages, I speculated that any one of them could be the culprit.

My first step was to try to replicate the problem locally, which fortunately I was able to do. I had been using Pygments in my *_config.yml* as the highlighter, despite the warning emails from Github Pages that they were using Rouge - it hadn't caused any issues up to this point. When I switched my local build to use the Rouge highlighter, the strange syntax highlighting showed up locally. 

Rouge was confirmed as the source of the problem when I took at look at the source code that was being generated and saw that the area in question was being wrapped in a `highlighter-rouge` class.[^1]

![highlighter-rouge class wrapping code](/public/assets/images/syntax-highlighting-inspector-broken.png)

Part of the issue here was that Pygments has a CFML lexer - Rouge does not - but since Rouge is the Github Pages highlighter, I needed to find a way to work with it. 

The lack of a CFML lexer lead me to believe that the real problem was with how Rouge was parsing my markdown. It didn't recognize the code, so the classes it was applying were not accurate. If I could tell Rouge not to apply specific syntax highlighting, I reasoned that that should resolve the problem. A little digging lead me to the [kramdown's configuration for highlighting with Rouge](http://kramdown.gettalong.org/syntax_highlighter/rouge.html). Within its syntax highlighting options, you can set a default language. So, I ended up with setup in my *_config.yml*:

```text	
highlighter: rouge
markdown: kramdown
kramdown:
	input: GFM
	hard_wrap: false
	syntax_highlighter: rouge
	syntax_highlighter_opts:
		default_lang: text
```
			
By defaulting the syntax highlighting to plain text, I was able to remove all of the highlighting classes that were being applied and have the code output cleanly:

![basic rouge highlighting without syntax classes](/public/assets/images/syntax-highlighting-inspector-fixed.png)

Now, obviously, the ideal setup would be to actuall output CFML with CFML syntax highlighting, so time permitting, I'm going to look into working on a CFML lexer for Rouge. Fingers crossed on that project.

<hr>

[^1]: In retrospect, I found that a partial culprit was my `syntax.css` file, which was applying the background color to the `class="error"` span tag. In fact, replacing the contents of my `syntax.css` with a more [Github flavored one](https://github.com/aahan/pygments-github-style/blob/master/github.css) resolved the problem visually. That is to say, the inline code appeared fine, though the underlying issue of how the syntax highlighting was being applied was not resolved.








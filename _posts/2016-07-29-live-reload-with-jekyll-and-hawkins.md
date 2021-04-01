---
published: true
title: Live Reload with Jeykll and Hawkins
layout: post
tags: [jekyll,hawkins,ruby,blogging]
---
While struggling to write a different post, I procrastinated by deciding that, to boost my productivity, I *needed* the preview of the post to live-reload. It was actually really easy to set up (and it's very cool to use). <!--more-->

___
**Update - 07/17/2019**: Admittedly, this is a bit late to the party, but I should note that the approach used here is no longer needed. With the [release of Jekyll 3.7](https://jekyllrb.com/news/2018/01/02/jekyll-3-7-0-released/) in January of 2018, LiveReload support was built into the core development server. You can use it via the command: `bundle exec jekyll serve --livereload`

___

## The Problem

I use [LightPaper](http://lightpaper.42squares.in/) to blog. It's the best Markdown editor I've found (and I've tried a lot of them[^1]). While its built-in preview functionality is very good, there are still times that LightPaper's internal Markdown renderer differs from the way kramdown parses/converts the file.[^2] I find these occasional discrepancies frustrating, and they're one of the reasons I prefer to just run `bundle exec jekyll serve` and use the preview supplied by Jekyll's local server when blogging. The downside of this approach is the need to refresh the page in order to see changes, so in order to avoid writing, I decided to track down a solution.

## The Solution: Hawkins

My initial search for "jekyll realtime refresh" lead to a bunch of dated posts about using Grunt, Gulp, Browsersyc, `guard-livereload`, a Chrome plugin, and/or some ruby scripts to make it happen; all of which seemed like a lot more work that I wanted to do.[^3] A revised Google search, that included the year, did the trick: "jekyll refresh on save 2016". Halfway down the results page, I saw a post on [talk.jekyllrb.com](https://talk.jekyllrb.com/) from this year: [A tool for automatic browser refreshes](https://talk.jekyllrb.com/t/a-tool-for-automatic-browser-refreshes/2150).

The tool, which is exactly what I was looking for, is [Hawkins](https://github.com/awood/hawkins), by [@awood](https://github.com/awood). What is Hawkins, exactly? To parrot the README:

> Hawkins is a Jekyll 3.1+ plugin that incorporates LiveReload into the Jekyll "serve" process.

And, as explained by @awood in the initial forum post:

> Hawkins is heavily inspired by guard-livereload and rack-livereload but it's entirely self-contained and integrated with Jekyll so it's easier to setup and use.

## Setting Up Hawkins

I followed the "How To Use" guide from the README and added the following to the Gemfile in the root of my project:

```ruby
group :jekyll_plugins do
  gem 'hawkins'
end
```
Then, from the root of the project, I ran `bundle install`, which downloaded and installed the plugin and all necessary dependencies. That's it; it's ready to use. The syntax for serving the preview with Hawkins is very similar to the standard Jekyll command:

```shell_session
$ bundle exec jekyll liveserve
```
You'll know it's working, because the console output will include the new line "LiveReload Server: 127.0.0.1:35729" (which is not where the site is served from, it's just for serving *livereload.js*.) You can then access the site at the usual: <http://127.0.0.1:4000>. When you do, the console will acknowledge it with the output: "LiveReload: Browser connected". When you save your Markdown post, the preview will refresh automatically:

![blogging with LightPaper, Jekyll, and LiveReload by Hawkins](/public/assets/images/hawkins-jekyll-lightpaper-preview.png)

Happy blogging!



<hr />
[^1]: Mou, MacDown, Byword, iA Writer, plugins for Sublime Text 3, to name a few. None of them bad; they're just not at quite the same level.
[^2]: Here are two basic example: 1) They handle escaping special characters differently, which can result in some very funky looking output. 2) LightPaper's syntax highlighting is not as robust.
[^3]: Update - 08/01/2016: In the comments, [@bdavidxyz](https://github.com/bdavidxyz) mentioned his project [Impatient Jekyll](http://bdavidxyz.github.io/impatient-jekyll/), which provides live-reload and more. If you'd like a more full featured dev environment for Jekyll, without that much more work, you should check it out.


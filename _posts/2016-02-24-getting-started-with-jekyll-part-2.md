---
date: 2016-02-24
published: true
title: Blogging with Jekyll (Part 2, Jekyll Setup)
layout: post
tags: [blogging,jekyll,ruby]
---
Yesterday I wrote about [getting started with Jekyll](/2016/02/23/getting-started-with-jekyll-part-1.html), which was really just recounting my difficulties getting Ruby set up and configured correctly. Once Ruby was configured and the Jekyll gem installed, I assumed that it would be a fairly straightforward process. It wasn't. But it probably should have been.<!--more-->[^1]

The blog had already been set up (via Tinypress.co) to publish on Github Pages, so it was already configured/structured/formatted using Jekyll. I cloned the repo locally and just wanted to be able to run a local version for writing, previewing, and development.

I went back to the [Jekyll Quick Start Guide](http://jekyllrb.com/docs/quickstart/) and skimmed it. I probably should have read a bit more carefully. I ran:

```shell-session
$ jekyll new . --force
$ jekyll serve
```

And, when I went to view the site locally, it had blown away the entire existing setup and posts, replacing them with the default Jekyll base installation. Oops. No harm though; I figured that running a hard reset on the repo would fix everything:

```shell-session
$ git fetch origin
$ git reset --hard origin/master
```

The site still wasn't showing correctly, possibly because the server instance was still running.[^2] The easiest route, it seemed, would just be to start over. From [their docs](http://jekyllrb.com/docs/usage/), I ran the command to kill the server instance: `ps aux | grep jekyll`, which was probably excessive, but got the job done. Then I deleted everything in the directory.

I pulled a clean copy of the blog repo again from Github, navigated to the directory, and ran `jekyll serve`, which got me a bit closer. The site was now showing correctly... except that the posts weren't being listed on the homepage. There was an error message in the terminal response:

```text
Message: "Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration file."
```

The first route I tried was to just install the gem:

```shell-session
$ gem install -n /usr/local/bin jekyll-paginate
$ jekyll build
$ jekyll serve
```

Unsurprisingly, given my track record at this point, this didn't work and I got the same error message. This is where I finally gain a small bit of common sense, and went back to the documentation, like I really, really should have from the start.

So, here are the docs: [Setting up your Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-pages-site-locally-with-jekyll/), which ultimately helped me get it up and running, after a few more hiccups, along the way. That's the [third part of this series](/2016/02/24/getting-started-with-jekyll-part-3.html).
<hr>

[^1]: To paraphrase: "The fault, dear reader, is not in Jekyll, / But in ourselves"

[^2]: A little digging after the fact seemed to confirm this: "Changes to _config.yml are not included during automatic regeneration." - From the [Jekyll Basic Usage Docs](http://jekyllrb.com/docs/usage/)

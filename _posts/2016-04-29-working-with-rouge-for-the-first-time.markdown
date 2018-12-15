---
published: true
title: Working With Rouge for the First Time
layout: post
tags: [ruby,rouge,jekyll]
---
When I started blogging with Jekyll, one of the projects that interested me was working on a CFML lexer for Rouge. It's been a while since I've written a post, so it seemed like a good time to just dive in, get my hands dirty, and see what happened. All I'm doing in this post is getting Rouge set up, so that I can start tinkering with it.

<!--more-->

I wasn't sure exactly where to begin, but I did anyway, which I've found to be an effective strategy for learning new tech. I forked the [Rouge repo](https://github.com/jneen/rouge) and then cloned it locally:

```shell_session
$ git clone git@github.com:mjclemente/rouge.git
```

There are instructions on the Rouge readme.md for [contributing](https://github.com/jneen/rouge#contributing), so I did my best to follow them. Because I had already [set up Ruby with rbenv](/2016/02/23/getting-started-with-jekyll-part-1.html), I was able to just run `bundle` to install the dev dependencies. That went smoothly, and the result was:

`Bundle complete! 10 Gemfile dependencies, 22 gems now installed.`

The next step was to run  `rake`, which, according to the docs, tests the core of Rouge. The tests ran successfully, and the result included this line:

```text
Run `rackup` and visit localhost:9292/:lexer_name to visually test a lexer.
```

So, I tried running `rackup` and it didn't work:

```text
-bash: rackup: command not found
```

I had to do a little digging to resolve this, but it seems like the issue was with my rbenv setup. I needed to manually run the `rbenv init` in order to get the new Gemfile dependencies, including `rackup`, added to rbenv's shims directory (and thereby accessible via my $PATH).[^1]

Once I restarted my terminal, the `rackup` command ran successfully, and I was able to visually access the lexers, like the one for html: http://localhost:9292/html

![rouge html lexer preview](/public/assets/images/rouge-html-lexer-syntax-highlight-preview.png)

That's it for today - Rouge set up, ready to dig in to how the lexers work.

<hr />

[^1]: Update (7/20/2016): For those running Cygwin, [Jakub Klimek](/2016/04/29/working-with-rouge-for-the-first-time.html#comment-2792924397) noted, in the comments, that he had to manually add rackup to PATH via his .bashrc: `export PATH=$PATH:~/.gem/ruby/gems/rack-1.6.4/bin/`

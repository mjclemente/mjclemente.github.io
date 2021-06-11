---
date: 2018-10-22
published: true
title: "TIL: The Other Way to Embed Images in Markdown"
layout: post
tags: [markdown,til]
---
This will be old news - very old news - for some people. Today I learned the "other" way to add images to a Markdown document; instead of writing them inline, you can embed images *reference-style*.
<!--more-->

I've always written Markdown images inline:

```markdown
![Alt Text](https://blog.mattclemente.com/public/500.png)
```

And while this works, the syntax can be a bit clunky - I don't find it easy to scan when writing a post or documentation - and honestly, I never remember it.

So, after years of writing Markdown, I was very surprised to learn that there is an alternative way of embedding images. Enter reference-style:

```markdown
![Alt Text][id]

<!-- etc. etc. etc. -->

[id]: https://blog.mattclemente.com/public/500.png
```

Using an reference identifier in place of the image path results in a cleaner, more succinct embedded image, as well as making it possible to manage and easily see all image references at the end of the document.

This is not new. If you thoroughly read documentation (which apparently I failed to do in this case), it's right there in [John Gruber's syntax guide](https://daringfireball.net/projects/markdown/syntax#img).

There are obviously tradeoffs to both approaches, and the only real guide is preference. So, if, like me, you were unaware of this option... now you know.

Also, for what it's worth, this approach [can also be used for links](https://daringfireball.net/projects/markdown/syntax#link).
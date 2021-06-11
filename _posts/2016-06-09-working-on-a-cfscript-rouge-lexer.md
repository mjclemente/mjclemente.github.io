---
date: 2016-06-09
published: true
title: Working on a CFScript Rouge Lexer
layout: post
tags: [ruby,rouge,jekyll]
---
Well, I submitted by PR for [adding CFScript](https://github.com/jneen/rouge/pull/492) to Rouge, so we'll see how that turns out. I only worked on CFScript - I don't have much use for tag highlighting, at present, and that kept the scope of the project more manageable. I was pretty happy with the result, though it did take more time than I had anticipated. CFML certainly has its quirks, and I even stumbled across a few operators that I was unaware of.
<!--more-->

My guide, throughout the process, was the [Using the lexer DSL](https://github.com/jneen/rouge#using-the-lexer-dsl) section of the Readme. Rather than providing play-by-play for the entire trial-and-error filled process, I'll just hit a few items of note:

## Regarding Adding Lexers to Rouge
There are 4 files needed when adding a new lexer to Rouge. They are:

1. The lexer itself. These are located in `/lib/rouge/lexers/lexername.rb`
2. The sample file. This is where you show everything that it can do, and demonstrate that it works in all different situations. Located in `/spec/visual/samples/lexername`.
3. The spec file, located in `/spec/lexers/lexername_spec.rb`. I honestly don't know exactly what this is for, but I think one of the purposes is to help Rouge determine what lexer to use, based on file characteristics.
4. The demo file, in `/lib/rouge/demos/lexername`. Again, not sure the exact purpose. Testing, perhaps. Most just hold a small portion of the sample file. 

## If Java and JavaScript Had a Child...

I started trying to build the CFScript lexer as an extension of the JavaScript lexer, which ultimately proved futile. While CFScript bears a number of similarities to JavaScript, it is not "based on JavaScript", and there were just too many structural and syntactical differences. It's really an interesting hybrid of Java and JavaScript, at least insofar as putting together a lexer is concerned, which I hadn't considered before - this seems like an effective way of explaining the language.

## Testing the Lexer

One of the biggest challenges was putting together the sample/testing document. I needed to find examples of all of the data structures, operators, and syntax variation of the language, which proved difficult. It wasn't until I reached out to [@ryaneberly](https://github.com/ryaneberly) of [cfparser](https://github.com/cfparser/cfparser) and he pointed me to [their tests](https://github.com/cfparser/cfparser/tree/master/cfml.parsing/src/test/resources/cfml/tests) that I found a comprehensive set of examples. I really needed to get the sample code in place, before I could properly work on the lexer itself, so this was invaluable. It also revealed a significant number of edge cases that I hadn't considered. 

My basic, guiding principle, was that the CFScript Rouge lexer should match the output of the existing [ColdFusion CFC Pygments lexer](http://pygments.org/docs/lexers/?highlight=coldfusion#pygments.lexers.templates.ColdfusionCFCLexer), for the same block of code[^1]. There where a few instances, where I think I improved on the Pygments lexer, but overall they're nearly identical, and it proved a helpful measure for the success of the project.

## CFML: Mime-Type and "New" Operators

Insofar as I can tell (and unlike most of the other languages for which Rouge has lexers), there are no particular mime-types associated with .cfm or .cfc files. I also don't know what the significance of this is - if you know, I'd love to be enlightened.

With regard to the operators supported in CFScript[^2], the first surprise was the number of actual text phrases that can be used. For example, `does not contain` or `greater than or equal to` are both valid, within a script statement. Why someone would use the latter, literally spelling it out, instead of the more ubiquitous and succinct `>=` is beyond me. Additionally, I encountered, for the first time, the following "Boolean" operators:

*  `XOR`: **Exclusive or** - Return True if one of the values is True and the other is False. Return False if both arguments are True or both are False. For example, `True XOR True` is False, but `True XOR False` is True.
* `EQV`: **Equivalence** - Return True if both operands are True or both are False. The EQV operator is the opposite of the XOR operator. For example, `True EQV True` is True, but `True EQV False` is False.
* `IMP`: **Implication** - The statement `A IMP B` is the equivalent of the logical statement "If A Then B." `A IMP B` is False only if A is True and B is False. It is True in all other cases.

I'm not sure if/when I'd use these, but it's good to know they exist, and I'll certainly be keeping them in mind. If I stumble across a use case, I'll post about it - if anyone knows of a use case, I'd love to learn it.

So here's a screenshot of the final result:

![rouge cfscript lexer preview](/public/assets/images/rouge-cfscript-lexer-syntax-highlight-preview.png)

I'll be sure to post if/when the PR gets merged - I'm looking forward to getting the code highlighted correctly on this blog!

<hr />
[^1]: Here's the [Pygments lexing of my sample code](http://pygments.org/demo/5189344/)
[^2]: Here's the [most recent documentation on the CFScript operators](https://helpx.adobe.com/coldfusion/developing-applications/the-cfml-programming-language/using-expressions-and-number-signs/expressions-developing-guide.html).

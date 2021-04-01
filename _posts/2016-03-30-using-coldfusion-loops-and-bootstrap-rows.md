---
published: true
title: Using ColdFusion Loops with Bootstrap Rows and Columns
layout: post
tags: [coldfusion,cfml,bootstrap,loops]
---
One of the most common application requirements is outputting an arbitrary amount of data in organized rows and columns. It used to be done with tables, now it's done with Bootstrap; it looks better, but the principle is the same: output items until the row is full, then start a new one - and make sure you close all the tags that you open, in the correct order. <!--more-->

I've handled this in a variety of ways in the past; sometimes with a variety of incrementing counters and `<cfif>` tags, occasionally with `<cfoutput group="someCol">`. It always ended up cluttering the view and it never felt clean. While every situation is different, I recently hit upon a simple solution for outputting data in Bootstrap's rows and columns, using ColdFusion's [ternary](http://www.bennadel.com/blog/1643-learning-coldfusion-9-the-ternary-operator.htm) and [mod](http://www.bennadel.com/blog/665-and-on-the-seventh-row-mod-created-1-and-it-was-good.htm) operators (of course those links are to Ben Nadel's blog). 

Here was the situation: I needed to output data in four columns, with as many rows as necessary. Rather than beating around the bush, here's the code[^1]:

```text
<cfloop index="loopIndex" from="1" to="#arrayOfData.len()#">
	#(loopIndex MOD 4 EQ 1) ? '<div class="row">' : ''#
		<div class="col-sm-3">
			<!---Col Content Here--->
		</div>
	#(loopIndex MOD 4 EQ 0 OR loopIndex EQ arrayOfData.len()) ? '</div>' : ''#
</cfloop>
```

Just a few notes on it:

1. We're using ColdFusion 11, so it uses the very convenient `array.len()` syntax, instead of `arraylen(array)`.
2. We're outputting 4 items per row, which is why it's `MOD 4`, and why Bootstrap's 12 grid columns are broken down into `col-sm-3`.
3. You'll see that the logic of the opening and closing `mod` operations are based on 1 and 0, respectively, based on whether it should be the opening or closing of the row div. 
4. Additionally, the closing `mod` also needs to account for closing the row if it is the last item in the array. 

So, nothing groundbreaking here, but I was pleased with how clean the resulting code was. I'm sure there are circumstances that will require additional logic and manipulation, but this served my purpose - and I think it would provide the basis of my approach for more complex situations.

<hr>

[^1]: Here's a gist where you can see the code: [Loop with Ternary and Mod Operators](http://trycf.com/gist/a6d123332b7bce30fd53a1beb5de90b8/acf11?theme=monokai). Granted, the formatting, which is the whole point, won't be displayed properly, because Bootstrap isn't used for the output.
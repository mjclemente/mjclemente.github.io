---
published: true
title: "TIL: ucFirst() for Word Casing in Lucee CFML"
layout: post
tags: [coldfusion, til,ucfirst]
---
Despite 10+ years of CFML programming, I'm still stumbling across new functions. Today, it was the awkwardly named but surprisingly handy `ucFirst`, a Lucee-specific function for dealing with string capitalization.
<!--more-->

I'll just quote [the docs](https://cfdocs.org/ucfirst), regarding what `ucFirst` does:

> Transforms the first letter of a string to uppercase or the first letter of each word, and optionally lowercase uppercase characters.

Admittedly, that's a bit of a mishmash, with three different modes of functionality:

- Without any options, `ucFirst` transforms a string into "sentence case." That is, it ensures the first letter is capitalized. This is also the casing of proper nouns, like first and last names, which is what I needed - a bit more on that at the end.

  ```cfc
  ucFirst( "the time has come.");
  // The time has come.
  
  ucFirst( "maggie");
  // Maggie
  ```

- You can convert a string to "title case" by setting the second argument, `doAll`, to `true`:

  ```cfc
ucFirst( "alice's adventures in wonderland", true );
  // Alice's Adventures In Wonderland
  ```
  
- The third optional argument is `doLowerIfAllUppercase`. Setting it to `true` only impacts the result if the *entire string* is uppercase:

  ```cfc
  ucFirst( "I AM YELLING ONLINE!", false, true);
  // I am yelling online!
  
  ucFirst( "Frank McCourt", false, true );
  // Frank McCourt
  ```

Now, I'll be this first to admit that this is not mind-blowing stuff - but it's just what I was looking for.

I was trying to correct user-provided names that were, in many *cases* (pun-intended), improperly cased; e.g. *BRENDA FLOR*, *rafael Kerr*, and other variations. A little Googling to find a solution showed that this isn't an uncommon problem:

- [Capitalize first letter of first word in a every sentence in ColdFusion](https://stackoverflow.com/questions/23808425/capitalize-first-letter-of-first-word-in-a-every-sentence-in-coldfusion) (Stack Overflow)
- [Capitalize the first letter?](https://community.adobe.com/t5/coldfusion/capitalize-the-first-letter/td-p/419701) (Adobe Support Community)
- [Capitalize the first letter in every word using ColdFusion - Regex to the rescue](https://www.n-smith.com/index.cfm/my-blog/2012-04-17-capitalize-start-of-each-word-using-coldfusion/)
- [CapFirst()](https://cflib.org/udf/CapFirst) (CFLib.org)
- [upperFirst()](https://cflib.org/udf/upperFirst) (CFLib.org)

As you can see, there are a range of solutions, with varying degrees of control, complexity, and accuracy. Your mileage will vary, depending on the ColdFusion engine that you're using and your application's requirements (Don't forget to [account for Celtic names!](https://blog.adamcameron.me/2014/04/regex-for-simplifying-string.html))

If you're using Lucee, nothing beats `ucFirst` for simplicity, as it's built in. And while there's no guarantee that its output is 100% correct, in our particular use case, it was accurate enough:

```cfc
newFirstName = ucFirst( firstName, true, true );
newLastName = ucFirst( lastName, true, true );
```

So, there you go - another tool that might come in handy, should the need arise.




---
date: 2019-06-14
published: true
title: "TIL: Compound Assignment Operators in CFML (+= and more)"
layout: post
tags: [java, cfml,til]
---
Okay, so this is slightly embarrassing, in that I'm writing about a "feature" that was added in ColdFusion 8... so, over 11 years ago. I'm talking about the compound assignment operators: `+=`, `-=`, `*=`, `/=`, and `%=`.
<!--more-->

To be clear, I wasn't entirely unaware of this functionality. But while I'd used `+=` before, it just recently came to my attention that this syntax extends to other mathematical operators, including multiplication and division.[^1]

To illustrate the functionality I'm referring to, here's some code, rather than a long-winded explanation:

```cfc
value = 5;
// The value is initially set to 5

value++; // = 6
// increments by 1

value--; // = 5
// decrements by 1

value += 5; // = 10
// variable set to itself plus operand (value on right)

value *= 5; // = 50
// variable set to itself multiplied by operand

value -= 5; // = 45
// variable set to itself minus operand

value /= 5; // = 9
// variable set to itself divided by operand

value %= 5; // = 4
// variable set to the remainder of itself divided by operand
```

That's it. Instead of needing to write, for example, `value = value * 10`, you can use the compound assignment operator shorthand `value *= 10`. 

Brief digression: In case explanation is needed, that last example is using `%`, which is the *modulus operator*. It's interesting, handy, and merits further examination, but is outside the scope of this post. 

Now, you might be asking yourself, what's the use case for this? There is none. You never *need* to use a compound assignment operator. It's just a shortcut. My day-to-day programming doesn't involve a lot of numeric calculations, but if yours does, you may find this succinct syntax convenient. At the least, it's another potentially handy widget in your programming toolbox and you'll know what it means if you run across it in someone else's code.



___
[^1]:It's not like this is a hidden feature either; it's right there in the docs, under [Arithmetic operators](https://helpx.adobe.com/coldfusion/developing-applications/the-cfml-programming-language/using-expressions-and-number-signs/expressions-developing-guide.html#Arithmeticoperators).
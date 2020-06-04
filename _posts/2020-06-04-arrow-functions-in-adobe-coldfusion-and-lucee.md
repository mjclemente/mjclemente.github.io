---
published: true
title: "Fat Arrow Function Support in Lucee 5 and Adobe ColdFusion 2018"
layout: post
tags: [arrow functions,cfml,coldfusion]
---
Arrow functions have been around in CFML since Lucee 5 and Adobe ColdFusion 2018 (Update 5), respectively. Despite this, I only recently began trying to get comfortable with the syntax. I was surprised to find that, while Lucee added fat arrow support much earlier, Adobe ColdFusion provides more complete support for the syntax.
<!--more-->

## On the Arrow Function Syntax

Whether you like its succint nature, or find it off-putting, the arrow function syntax is here to stay. I believe it's worth learning, even if you don't intend to use it; otherwise, the arrow functions you stumble upon in someone else's code will be an indecipherable mess of parenthesis and greater than signs.

I don't intend this post to be a primer on (fat) arrow functions. For that, I'd recommend:

- [Ben Nadel on Fat-Arrow Functions](https://www.bennadel.com/blog/3648-fat-arrow-functions-and-lambda-expressions-are-supported-in-lucee-5-3-2-77.htm) (ColdFusion primer)
- [Adobe ColdFusion documentation on Lambdas](https://helpx.adobe.com/coldfusion/developing-applications/the-cfml-programming-language/extending-coldfusion-pages-with-cfml-scripting/using-closures.html#lambda) (official docs)
- [MDN docs on Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) (examples - while JavaScript, still helpful)
- [Fat Arrow Functions in CFML](https://www.youtube.com/watch?v=bxblJ_PDuwo) (my live-coding exploration of the topic)

## Engine Differences (Lucee 5 vs. Adobe ColdFusion 2018)

As I mentioned at the top, Lucee introduced support for arrow functions in Lucee 5, which was released in November of 2016. It wasn't until September 2019, with update 5 to ColdFusion 2018, that Adobe also supported the syntax. However, as you'll see, ACF supports a wider variety of succinct arrow function syntaxes.

### Fat Arrow Syntax Supported by Both Engines

The following basic examples, some of which were cribbed and slightly modified from Ben's blog post above, are supported by both Lucee 5 and ACF 2018.

#### Basic

```cfc
greet = ( name ) => {
  return( "Hello, #name#." );
};

writeOutput( greet( "Matthew" ) ); // Hello, Matthew.
```

#### Dropping `return` and curly brackets

```cfc
// When the only statement is `return`
// we can remove `return` and the surrounding curly brackets
depart = (name) => "Goodbye, #name#.";

writeOutput( depart( "Marcy" ) ); // Goodbye, Marcy.
```

#### Without parameters

```cfc
shrug = () => "¯\_(ツ)_/¯";

writeOutput( shrug() ); // ¯\_(ツ)_/¯
```

#### Within iterators

Examples of iterators include the array `.map()` member function, `arrayMap()`, and similar.

```js
cuts = [ "julienne", "chiffonade", "dice" ];
upperCuts = cuts.map( 
  ( item, index ) => {
    return item.ucase();
  }
);

writeDump( var='#upperCuts#', abort='true' );
// ["JULIENNE","CHIFFONADE","DICE"]
```



### Syntax Supported only by Adobe ColdFusion 2018

The final three examples work, as they should, in Adobe ColdFusion 2018, but result in a syntax error in Lucee. I found this a bit disappointing; hopefully this incompatibility/shortcoming is addressed soon.

#### Without parentheses

```js
// Parentheses are optional when there's only one parameter name
greet = name => "Hello, #name#.";

writeOutput( greet( "Matthew" ) ); // Hello, Matthew.
```

#### Without `return` and curly brackets, within iterators

```js
cuts = [ "julienne", "chiffonade", "dice" ];
upperCuts = cuts.map( 
  ( item, index ) => item.ucase(); 
);

writeDump( var='#upperCuts#', abort='true' );
// ["JULIENNE","CHIFFONADE","DICE"]
```

#### Shorthand, within iterators

```js
cuts = [ "julienne", "chiffonade", "dice" ];
lengths = cuts.map(cut => cut.len());

writeDump( var='#lengths#', abort='true' ); // [8,10,4]
```

Obviously, there are likely variations and edge cases that I haven't covered. If you find one, let me know in the comments.

## Lucee Issues

There are a number of Lucee tickets related to this. Here's one, for example: [LDEV-2417](https://luceeserver.atlassian.net/browse/LDEV-2417). Brad Wood also put together a [Github repo demonstrating the incompatibilities](https://github.com/bdw429s/CFML-lambda-fat-arrow-functions). So, if this is something that interests you, make it known on those tickets!

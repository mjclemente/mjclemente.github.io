---
date: 2019-07-12
published: true
title: "What is the Modulus Operator? A Short Guide with Practical Use Cases"
layout: post
tags: [modulus operator, til, modulo, operators]
redirect_from: "/2019/06/16/modulus-operator-modulo-operation.html"
---
Addition, subtraction, multiplication, and division. These are the four mathematical operations I was taught during my childhood education, and their operators, `+`, `-`, `*`, `/`, are very familiar. I was not taught `%`,  the ***modulus operator***, which I recently discovered can be quite useful and interesting in its own right.

<!--more-->

The modulus operator, written in most programming languages as `%` or `mod`, performs what is known as the [modulo operation](https://en.wikipedia.org/wiki/Modulo_operation). You next response, understandably, might be, "That doesn't clarify anything," so let's take a closer look:

* TOC
{:toc}
## How It Works

The modulus operator - or more precisely, the modulo operation -  is a way to determine the *remainder* of a division operation. Instead of returning the result of the division, the modulo operation returns the whole number remainder.

Some examples may help illustrate this, as it's not necessarily intuitive the first time you encounter it:

```
5 % 1 = 0
// 5 divided by 1 equals 5, with a remainder of 0

5 % 2 = 1
// 5 divided by 2 equals 2, with a remainder of 1

5 % 3 = 2
// 5 divided by 3 equals 1, with a remainder of 2

5 % 4 = 1
// 5 divided by 4 equals 1, with a remainder of 1

5 % 5 = 0
// 5 divided by 5 equals 1, with a remainder of 0
```

It may be helpful to think back to your early math lessons, before you learned fractions and decimals. Mathematics with whole numbers behaves differently - when dividing numbers that aren't even multiples, there's always some amount left over. That remainder is what the modulo operation returns.

If this seems strange, boring, or not particularly useful, bear with me a bit longer - or just skip ahead to the [use cases](#use-cases). 

### The Modulo Operation Expressed As a Formula

As one final means of explication, for those more mathematically inclined, here's a formula that describes the modulo operation: 

```
a - (n * floor(a/n))
```

By substituting values, we can see how the modulo operation works in practice:

```
100 % 7 = 2
// a = 100, n = 7
100 - (7 * floor(100/7)) = 2
```

If you don't find the formula helpful, don't worry - I didn't either at first. Some people find this abstract representation helps deepen or clarify their understanding of the operation, but you don't need to know it.

A final note here - if you're wondering how the modulo operation functions with negative numbers or decimals, that's a bit outside the scope of this article. For our purposes here, we'll only be dealing with positive integers.[^1]

Okay, enough with the math for now. While returning the remainder is what the modulo operation does, that's not its only use; indeed we'll see that it's handy for a good deal more - but that was a necessary starting point. 

## Use Cases for the Modulo Operation

When I first encountered the it, the modulus operator seemed little more than a bit of mathematical trivia. I found it far more interesting as I started to learn its practical utility. I'll discuss a few applications here.

### Even / Odd and Alternating

One of the most basic use cases for the modulus operator is to determine if a number is even or odd.[^2] This is possible because `x % 2` always returns either 0 or 1. Even numbers, because they are evenly divisible by 2, always return 0, while odd numbers always return the remainder of 1. Here's what I mean:

```
for( i=0; i <= 10; i++ ) {
  writeOutput( i%2 );
}
// 01010101010
```

So, what's the use case? This technique is often used to alternate values within a loop. The first time I used the modulus operator, it was to manually zebra-stripe table rows, with the row's background color based on whether it was even or odd. In a similar vein, I've used `%` to distribute the results of a web-form to two recipients, on an every-other basis; in pseudocode:

```
if( incrementalId % 2 == 1 )
  // send to one
else
  // send to the other
```

It's a convenient hack any time you have a group or stream of records, widgets, leads, etc., that you want to handle on an alternating basis.

### Restrict Number to Range

When you're using the modulus operator for even/odd alternation, you're actually taking advantage of one of its more helpful properties, though you might not realize it. Here's the property: **the range of `x % n` is between 0 and `n - 1`**, which is to say that *the modulo operation will not return more than the divisor*.[^3]

Again, examples might help clarify this idea; in each instance here, the divisor is 5, so results will range from 0 - 4.

```
1 % 5 = 1
// 1 cannot be divided by 5, so the remainder is 1

4 % 5 = 4
// 4 cannot be divided by 5, so the remainder is 4

7 % 5 = 2
// 7 divided by 5 equals 1, with a remainder of 2

25 % 5 = 0
// 25 divided by 5 equals 5, with a remainder of 0

218 % 5 = 3
// 218 divided by 5 equals 43, with a remainder of 3
```

As you can see, regardless of the initial number, the modulus (divisor) limits the range of the result. 

On its own, this property doesn't seem useful, but when applied within a larger set, it can be used to create a pattern of circular repetition. Here's an example of incrementing numbers with a modulus of 3:

```
0 % 3 = 0
1 % 3 = 1
2 % 3 = 2
3 % 3 = 0 // cycle back to 0
4 % 3 = 1
5 % 3 = 2
6 % 3 = 0 // cycle back to 0
```

Notice how the result of the modulo operation keeps repeating 0-1-2; the values wrap around. One way to describe these results is as a [circular array](https://www.quora.com/What-is-a-circular-array-and-how-does-it-work), like numbers on the face of a clock. Let's take a closer look at a practical application of this property.

#### Rotating Through Limited Options (Circular Array)

In a situation with a limited number of options - weekdays, primary colors, company projects, clients, etc. - we can use the modulo operation to walk through them in a repeating loop. That is, we can treat the array of options as a circle that we just keep cycling through. 

Here's a somewhat contrived example to illustrate this:

```
// array of options that we want to cycle through
weekdays = [ 'Mon', 'Tue', 'Wed', 'Thu', 'Fri' ];

// option count provides modulus (divisor)
dayCount = weekdays.len();

employeeCount = 14;

// loop over employees while rotating through days
for( i=0; i < employeeCount; i++ ) {

  // employee number mod option count
  dayIndex = i % dayCount;
  // adjust because CFML array indexed from 1
  dayIndex++;
  // use result to cycle through weekday array positions
  weekday = weekdays[ dayIndex ];

  writeOutput( "Scheduling employee on #weekday#. " );
}
```

As we step through a list of employees, we assign each to a weekday. Once we've scheduled five employees, we reach Friday. With no more options, assignment then loops back to Monday and continues the cycle until all employees are scheduled. This is possible because, as shown in the previous section, the modulus of 5 (the number of weekdays) returns a circular array of 0-4, that we can map to options in our weekday array. Seeing and understanding this was a major revelation for me.

### Every Nth Operations in a Loop

Another application of the modulo operation is determining an interval within a loop; that is, calculating occurrences such as "every fourth time," "once every ten," etc. 

The principle, in this case, is that if `n` is an even multiple of `x`, then `x % n = 0`. Consequently, `x % 4` will return 0 every fourth time; `x % 10` will return 0 every tenth time, and so on. One practical use of this is providing feedback within long or long running loops:

```
// given a list of widgets, files, people, etc.
longList = 10000; 
feedbackInterval = 100; // to be used as the modulus

// loop over the list to process each item
for( i=1; i <= longList; i++ ) {
	
  // perform some operation
	
  // mod operation gives feedback once every hundred loops
  if( i % feedbackInterval == 0 ) {
    percentCompleted = ( i / longList ) * 100;
    writeOutput( "#percentCompleted# percent complete. " );
  }
	
}
```

You could use a similar principle to trigger client/user interaction based on time or engagement. For example, send a promo every 90 days, or offer a feedback survey every 50 logins. 

Keep in mind that time is also, in effect, a long running loop. Every hour has 60 minutes, repeating on a cycle. If you wanted to schedule a task to run four times an hour, you could achieve this using the modulo operation - just run it when `minutes % 4 == 0`.

### Converting Units of Measure

Converting units of measure is common example of the modulo operation's practical utility. The general use case is when you want to convert a smaller unit, such as minutes or inches/centimeters, to a larger unit, like hours or miles/kilometers; in these situations, decimals or fractions aren't always helpful. 

For example, if we wanted to know the number of hours in 349 minutes, having the result expressed as *5 hours 49 minutes* may be more helpful than *5.8167 hours*. Here's a quick-and-dirty take at what that type of conversion function might look like:

```
function minutesToHours( m ) {
  hours = floor( m/60 );
  minutes = m%60;
    
  return "#hours# hours #minutes# minutes";
}
writeOutput( minutesToHours( 349 ) );
```

Standard division (rounded down to the nearest integer) determines the number of hours, while the modulo operation is used to keep track of the remaining minutes. Whether you're dealing with time, distance, pressure, energy, or data storage, you can use this general approach for unit conversion. 

### Miscellany

You might think that I've exhausted all the situations in which you might use the modulus operator, but you'd be wrong. Here are a handful more that I found on Stack Overflow, Quora, and the internet at large:

- [Reversing a number](https://stackoverflow.com/a/42234911/5361034)
- [Converting linear data to a matrix](https://stackoverflow.com/a/37449343/5361034)
- [Determining if arrays are rotated versions of each other](https://stackoverflow.com/a/51461187/5361034)
- [Pagination](https://www.quora.com/What-would-be-a-practical-use-of-modulus-operator/answer/Graham-Cox-18)
- [Leap year calculation](https://www.quora.com/In-which-cases-is-the-Modulo-operation-used-in-programming/answer/Mesam-Zahid)

Additionally, once you're comfortable with the modulo operation, you shouldn't have any trouble solving the [FizzBuzz Question](https://blog.codinghorror.com/why-cant-programmers-program/) discussed here.

## Some Helpful Reading

Credit where it's due - I learned a lot from these posts while I was researching and writing this article:

- [The Modulo Operator in Java](https://www.baeldung.com/modulo-java)
- [How to Use the Modulo Operator in PHP](https://code.tutsplus.com/tutorials/how-to-use-the-modulo-operator-in-php--cms-32393)
- [Fun With Modular Arithmetic](https://betterexplained.com/articles/fun-with-modular-arithmetic/)
- [Recognizing when to use the modulus operator](https://stackoverflow.com/questions/2609315/recognizing-when-to-use-the-modulus-operator) (Stack Overflow)
- [In which cases is the Modulo (%) operation used in programming?](https://www.quora.com/In-which-cases-is-the-Modulo-operation-used-in-programming) (Quora)

Now you know it; go use it! If you've got other uses or examples that you'd like to share, I'd love to hear them.

___
[^1]:Programming languages vary in their approach to supporting and handing *floating point* (decimal) modulo operations, as well as negative numbers. Working with these is outside the scope of this article, my concern, and most practical use cases that I've read about or encountered.
[^2]:Lest I get called out in the comments, I should note that *bitwise operators* offer another, even more efficient approach to determining if a number is even or odd, but that's another topic for another day.
[^3]:I should note that the divisor in a modulo operation is also called the modulus.
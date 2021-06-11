---
date: 2017-08-20
published: true
title: Using the Builder Pattern in ColdFusion - Concrete Examples from SendGrid.cfc
layout: post
tags: [coldfusion,cfml,sendgrid.cfc,api,design patterns,builder pattern]
---
My latest side project has been developing [SendGrid.cfc](https://github.com/mjclemente/sendgrid.cfc) a CFML wrapper for the [SendGrid API (v3)](https://sendgrid.com/docs/API_Reference/api_v3.html). This post isn't about the API or SendGrid.cfc though; it's about the Builder Pattern. Apparently this approach to handling complex objects is well-known in the Java world, but I had not encountered it in ColdFusion before. So, if you're new to this design pattern (like I was), or you just want to see a real-world application of it, read on...
<!--more-->

## The Challenge: The Request Body POST to `/mail/send`

I've built a handful of ColdFusion API wrappers, but SendGrid's endpoint for sending emails is different from anything I'd previously encountered, both in terms of the complexity and size of the parameters it accepts. 

For example, when sending a message, there is no top-level `to` parameter. In order to send a message, even if there is only one recipient, you need to use the `personalizations` parameter, which is an array of structs. Each personalization struct contains a `to` key, which is also... an array of structs. Structs within the `to` array contain an `email` and `name` key; that is where the email address is actually set. 

Did you follow that? In order to send a message to an email address, you need to create a personalization array, with a struct that contains a `to` key, which is an array, containing another struct that contains an `email` key... this is what addressing the message `to` a single email address looks like:

```json
{
  "personalizations": [
    {
      "to": [
        {
          "email": "test@example.com",
          "name": ""
        }
      ]
    }
  ]
}
``` 
Also worth noting; those array fields can contain up to 1,000 items. And that's just the size/complexity with regard to the person(s) the message is addressed to; there are an additional 70-something parameters that can be set within the request body.

Let me be clear, this complexity isn't necessarily a bad thing; it enables extremely robust personalization of messages, with very granular controls. 

The challenge, in building this wrapper, was to design as simple and intuitive an interface as possible for constructing a very complex object; I had no idea how to do it. After digging into SendGrid's official APIs, I came across a solution.

## The Solution: The Builder Pattern (with a fluent-ish interface)

The Builder Pattern, which I was not familiar with before this project, is an approach to constructing complex data structures one piece at a time. The builder object takes each parameter, handles the internal logic of setting it, and then returns the full object, which enables method chaining. Once all the parameters have been set, the builder is able to *build* the intended data structure. A *fluent interface* means that the methods that you pass the parameters to are named in such a way that they almost read like standard prose.

Enough with the abstract. Show me!

## The Builder Object: *helpers/mail.cfc*

*Mail.cfc* is an object intended to make constructing SendGrid's JSON easy. Here's an example:

```cfc
mail = new helpers.mail()
	.from( 'you@example.com' )
	.replyTo( 'No Reply <noreply@example.com>')
	.subject( 'Sending with SendGrid is Fun' )
	.to( 'friend@example.com' )
	.plainFromHtml( '<p>and <b>easy</b> to do anywhere, even with <em>ColdFusion</em></p>');
```
A few things worth noting here:

* The methods can be chained, enabling the object to be constructed in a single list of linked commands, which assemble it one piece at a time. 
* The methods also abstract away the potential complexity of setting a parameter. For example, the `to()` method can simply accept the email address as a string, and it handles the rest of the work. All the work with the `personalizations` is done behind the scenes.
* The method names (for the most part) convey the intent of the code in a readable manner. The only method that might need clarification here is `plainFromHtml()`, which takes the email's HTML content and uses it to also generate plaintext content. 
* Within the builder object there are additional methods available for the various other parameters that can be set, as well as to extend the elements in each array parameter.

The builder object can then construct the request body for SendGrid's API by calling `build()`. Here's the resulting JSON:

```json
{
  "from": {
    "email": "you@example.com",
    "name": ""
  },
  "reply_to": {
    "email": "noreply@example.com",
    "name": "No Reply"
  },
  "subject": "Sending with SendGrid is Fun",
  "personalizations": [
    {
      "to": [
        {
          "email": "friend@example.com",
          "name": ""
        }
      ]
    }
  ],
  "content": [
    {
      "value": "\rand easy to do anywhere, even with ColdFusion",
      "type": "text/plain"
    },
    {
      "value": "<p>and <b>easy</b> to do anywhere, even with <em>ColdFusion</em></p>",
      "type": "text/html"
    }
  ]
}
```
The result is verbose, but generating it is trivial; in this example, just five readable, chainable methods. Honestly, I was really impressed with how well the Builder Pattern worked in this case, making the complexity manageable by tackling it one step at a time.

Disclaimer: *Obviously the Builder Pattern is not the solution for every problem. It might not fit yours as well as it fit mine.*

## Chaining Methods in ColdFusion

Admission time; prior to my work on SendGrid.cfc, I had never set up a CFC with methods that could be chained. I didn't know how to set one up. I didn't even know if it was possible. 

The most significant realization I had while working on this project was that CFC method chaining is possible, easy, and can be a very effective design strategy, as it enables use of the Builder Pattern. 

In ColdFusion, you can utilize method chaining by returning `this` from the methods in a CFC. Here's some pseudo-code for a chainable method:

```cfc
  public any function handleSomeParameter( required any value ) {
    //blah, blah, blah, do it
    return this;
  }
```
And here's the actual `to()` method from *mail.cfc*, employing the same approach:

```cfc
  public any function from( required any email ) {
    setFrom( parseEmail( email ) );
    return this;
  }
```
This paradigm changed and widened the way that I thought about object creation and data manipulation. In the past, if I had to set a CFC property, I might return `void`, a boolean indicating success, or maybe some processed data, based on the parameter. These approaches all have their place, but if you want to employ the Builder Pattern and chain your methods, you return the complete CFC.

Now, I'm not going to run around returning `this` from all my methods (and you shouldn't either), but it's another tool that I hope you find as helpful as I did.

Finally, if you want to use ColdFusion to send emails with SendGrid, check out [SendGrid.cfc](https://github.com/mjclemente/sendgrid.cfc); development is ongoing, and I'm doing my best to keep the documentation clear and complete.

<hr />
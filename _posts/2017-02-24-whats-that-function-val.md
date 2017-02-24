---
published: true
title: What's that function - val()?
layout: post
tags: [coldfusion,what's that function]
---
What's the name of that function? The one that converts the opening numbers of a string into a numeric value? I always forget. For the record, it's `val()`.

<!--more-->
I probably always forget it because the name is... not particularly helpful, memorable, or clear. Here are the [official Adobe docs](https://helpx.adobe.com/coldfusion/cfml-reference/coldfusion-functions/functions-t-z/val.html) and, for runnable examples, check out [cfdocs.org/val](http://cfdocs.org/val).

So, why was I trying to remember it; that is, what good is `val()`? The most common use I have had for it is a convenient method for dealing with the status codes returned from http requests. The `statusCode` response from `cfhttp` requests comprised of two parts:

> The HTTP status_code header value followed by the HTTP Explanation header value; for example, "200 OK".

When dealing with RESTful APIs, generally I'm basing decisions on the numeric value returned from your requests; the HTTP explanation is just cruft. So, to that end, on occasion I've found myself running something like the following:

```cfc
//ACF 11+
cfhttp( method="GET", url="http://blog.mattclemente.com/", result="result" );

//ACF <=10
//httpService = new http( url = "http://blog.mattclemente.com/", method = "GET" );
//result = httpService.send().getPrefix();

if ( val( result.Statuscode ) >= 400 ) {
//uh oh
}
```

`Val()` let's me know if we've got the expected success header, or if we need to do some error handling.

## Why this post might be pointless

 While writing this post, I realized that there is direct access to the numeric status code returned by the `cfhttp` calls. It's just not presented in a straightforward way in the docs; it's found in the `responseheader` portion of the response. So, the following would also work:
 
```cfc
if ( result.responseheader.status_code >= 400 ) {
//uh oh
}
```

While the `val()` approach is a bit more succinct, I don't find it particularly easy to read when scanning the code, so I might be changing my approach to handling HTTP responses. 

Regardless, `val()` is the function, so next time, hopefully, I won't forget it. 

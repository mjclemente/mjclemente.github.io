---
date: 2020-11-30
published: true
title: "TIL: The Timezone parameter in CFML Date/Time Functions"
layout: post
tags: [til,timezone,datetimeformat,cfml,coldfusion]
---
The applications I've built up to this point haven't needed to account for time zones (and for that I consider myself fairly fortunate), which is probably one of the reasons that I only just discovered that ColdFusion's date/time functions can accept a `timezone` parameter.
<!--more-->

Well, that's not exactly true. Both Adobe ColdFusion (10+) and Lucee have a [`dateTimeFormat`](https://cfdocs.org/datetimeformat) function, which can optionally accept a `timezone` parameter. Here's what it looks like:

```cfc
// writing this example at 6:42 AM
datetime = now();
mask = "mm/dd/yyyy HH:nn zzz";

east = datetime.dateTimeFormat(mask, "EST");
// 11/30/2020 06:42 EST

west = datetime.dateTimeFormat(mask, "PST");
// 11/25/2020 03:42 PST
```

A few things to note here:

- By switching the time zone from `EST` to `PST`, the time output is shifted 3 hours earlier. That was easy!
- We're using `zzz` as the mask for outputting the three letter time zone.[^1]
- We're performing this on a DateTime object, not a string. That's important. Datetime objects exist relative to UTC, so their time zone can be shifted when formatting. Strings are, well, just strings, so the `timezone` parameter doesn't have any impact on them. Here's an example of what I mean:

  ```cfc
  datetime = "11/30/2020 6:42";
  mask = "mm/dd/yyyy HH:nn zzz";
  
  east = dateTimeFormat(datetime, mask, "EST");
// 11/30/2020 06:42 EST
  
  west = dateTimeFormat(datetime, mask, "PST");
	// 11/30/2020 06:42 PST
	```

* In the above example, the `datetime` variable is a string. As a consequence, switching the `timezone` argument from `EST` to `PST` does not alter the time displayed - it is 06:42 in both cases - though the time zone abbreviation is changed. Also note also that we can't use `dateTimeFormat()` as a member function when operating on a string; that's only available for DateTime objects.

## Two More Helpful Time Zone Related Functions

Now there are two questions that you might be asking yourself. First, how do I create a DateTime object that I can use with `dateTimeFormat`? For this, you can use [`createDateTime`](https://cfdocs.org/createdatetime), as shown here:

```cfc
datetime = createDateTime(2020, 11, 30, 6, 42);
west = datetime.dateTimeFormat(mask, "PST");
// 11/30/2020 03:42 PST
```

Second, and perhaps more importantly... *what is the default time zone for these DateTime objects that we're creating*? That is, if you use `now()` or `createDateTime()`, what is the time zone that you're starting from? You can find this out by using another function that I just learned about: [`getTimezoneInfo`](https://cfdocs.org/gettimezoneinfo), which retrieves the time zone information for the machine it is running on. The specific information returned by Lucee and Adobe ColdFusion differs slightly, but both let you know the UTC time offset:

```cfc
writedump(GetTimeZoneInfo());
```

## Additional Lucee Support

While writing this, I also learned that Lucee supports the `timezone` parameter in nearly all of its [DateTime methods](https://docs.lucee.org/categories/datetime.html). So, you can explicitly set the time zone in [`dateFormat`](https://docs.lucee.org/reference/functions/dateformat.html), [`timeFormat`](https://docs.lucee.org/reference/objects/datetime/timeformat.html), [`createDateTime`](https://docs.lucee.org/reference/functions/createdate.html), and more. That's really helpful!

## Additional Resources

After writing this article, I found two other resources that cover the same topic. They're linked from the Lucee docs, but somehow I missed them:

* [Andrew Dixon on *Datetime Timezone Handling in Lucee CFML*](https://www.andrewdixon.co.uk/2019/05/25/datetime-timezone-handling-in-lucee-cfml/)
* [Michael Offner's video on *Lucee Time Zone Handling*](https://www.youtube.com/watch?v=aIggbT8f3ls)

Happy time-zoning!

___
[^1]: While technically I only need one `z`, I'm using three so that the mask and output more closely resemble each other. The date masks follow Java's [SimpleDateFormat](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html) class. When masking time zones, lower case `z` is used for time zone abbreviations, `zzzz` is used for the full time zone name, and `Z` produces the RFC 822 time zone; for me, `-0500`.
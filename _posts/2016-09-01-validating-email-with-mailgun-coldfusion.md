---
date: 2016-09-01
published: true
title: You Should Probably Be Validating Email with Mailgun
layout: post
tags: [coldfusion,cfml,mailgun]
---
The TL;DR of this post, which should be readily apparent from the title, is that [Mailgun](https://www.mailgun.com/) provides an excellent service for email validation. Most of us don't want or need to know the details of the RFCs; we want an easy way to validate email addresses, so we can get on with building our apps. <!--more-->

For CFML developers, that easy method is supposed to be ColdFusion's `isValid()` function, but it has some well documented shortcomings. There are a few UDFs and regex strings for tackling this problem, but they don't come without issues. And finally, there's no small number of developers who ascribe to the dictum: "The only way to validate an email address is to deliver a message to it". Feel free to disagree, but I think Mailgun provides a better way.

*Just to get this out of the way, I am in no way affiliated with Mailgun. None at all. I just think they provide a pretty great service for developers.[^1]* Below is a quick summary of this post; jump to the end if using Mailgun is all you care about.

* TOC
{:toc}

So, let's dive right in; you should probably use Mailgun for email validation if:

## You're Using *isValid()* on ColdFusion 9 or 10

There are some significant problems with the way `isValid()` is implemented for email addresses on ColdFusion 9 and 10[^2]. For instance, the RFCs[^3] list the following special characters as being acceptable in an email address:

```text
- ' +  _ ! $ & * = ^ ` | ~ # %  / ? { }
```

Of the nineteen characters on that list, ColdFusion 9/10 only recognize the first four as being valid: `- ' + _`.  Additionally:

* The RFCs allow for almost anything within quotes, but `isValid()` fails all email addresses with quotes.
* `isValid()` enforces a seven character limit on TLDs , so many of the [new gTLD's](https://newgtlds.icann.org/en/program-status/delegated-strings), such as *.technology*, fail validation.
* Leading and trailing whitespace is allowed (it shouldn't be).[^4]

The bottom line is that you certainly shouldn't be using `isValid()` for email validation on ColdFusion 9 or 10. But maybe you upgraded; problem solved, right? Maybe not. You should probably be using Mailgun:

## Even If You've Upgraded to ColdFusion 11 or 2016

The Adobe team fixed a lot of the problems with `isValid()` in the two latest versions of ColdFusion. The full range of special characters is now allowed; quotes aren't automatically rejected, and there is no TLD character limit. It's substantially better. Of the problems I'm aware of in ColdFusion 9/10, only 2 weren't resolved:

* Leading/trailing whitespace is still an issue (or, rather, isn't an issue, because it's still allowed)
* Adam Cameron had a [series of test cases](http://blog.adamcameron.me/2013/02/email-address-validation-1-in-series.html) for `isValid()`. All but one are now handled correctly; for whatever reason, commas are still not allowed within quotes. So ["adam,cameron"@gmail.com](mailto:"adam,cameron"@gmail.com), which is a syntactically valid email address, does not pass validation.

A couple of new issues with `isValid()` were introduced, though they are certainly edge cases:

* Domain names that begin/end with `.` or `-` should be failed, but are not.
* It doesn't catch extraneous `@` signs in the address, if they are prefixed with a `\`. For example, [matt\@clemente@mattclemente.com](mailto:matt\@clemente@mattclemente.com) is incorrectly said to be valid.

So, all in all, it's not terrible, though not bug free. If you're like me, you might have read a little, realized that ColdFusion's built in validation has some problems, and decided that you want to use something better. Maybe you settled on the [isEmail](http://cflib.org/udf/isEmail) UDF from cflib.org. Well, you should probably use Mailgun for email validation if:

## You're Using the *isEmail()* UDF from cflib.org

I mention this, because I unthinkingly used it for a long time. It was copy-and-pasted, assumed to be better than `isValid()`, and then never given a second thought. I assumed it had to be better - after all, [Ray Camden has mentioned using it a few times](https://www.raymondcamden.com/2014/07/21/ColdFusion-isValid-Email-and-new-TLDs/) and it's cropped up in [StackOverflow answers](http://stackoverflow.com/questions/28738109/valid-invalid-email-format-policy-difference-between-coldfusion-and-php) as recently as 2015.

Here's the thing, <http://cflib.org/udf/isEmail> is old. Very old. And just because it's listed on cflib.org doesn't mean it's better than ColdFusion's built-in functionality. When I (recently) actually examined its accuracy, I was surprised to find that its results were almost identical to ColdFusion 9/10. It gets nearly all the same things wrong; the exception being that *isEmail* handles leading/trailing whitespaces correctly. So, not a great approach to take.

Some argue that *isEmail* has another advantage: you can "see under the hood" and when it validates incorrectly, you can improve it yourself. This is true, but it's hardly ideal. Sure, there are some basic problems with the regex that could be corrected easily enough, like the way it handles TLDs, but really testing and ensuring its accuracy is a *lot* of work. I have enough to do, without having to worry about verifying and maintaining the accuracy of my email validation.

In fact, you should probably use Mailgun for email validation if:

## You're Rolling Your Own Regex

Email validation is difficult. Speaking from experience, it's easy to underestimate the complexity of what constitutes a valid email address. For a primer on this issue, I recommend "[I Knew How To Validate An Email Address Until I Read The RFC](http://haacked.com/archive/2007/08/21/i-knew-how-to-validate-an-email-address-until-i.aspx/)" by [@haacked](https://github.com/haacked){:target="_blank"}. In case you're thinking "yeah, but isn't he overthinking it," here are a two more examples illustrating the scope of the problem:

* The PHP *is_email()* function ([@dominicsayers](https://github.com/dominicsayers)) comes with [164 unit tests](http://isemail.info/_system/is_email/test/?all).
* There's a [Perl module for email validation](http://www.ex-parrot.com/~pdw/Mail-RFC822-Address.html) with a regex of 6,000+ characters. The RFC grammar is described as being "surprisingly complex".
* Here's a [StackOverflow post about email validation](http://stackoverflow.com/questions/201323/using-a-regular-expression-to-validate-an-email-address). It has 70 answers, an absurd number of comments, and even though it's from 7 year years ago, it was active within the past 2 weeks.

Suffice to say, the rabbit hole of email validation is deep, dark, and filled with angry and opinionated comments. For arguments sake, let's say that you found the perfect regex for email validation (or, more realistically, you've found one that works 99% of the time, or even one that you consider "good enough"). I sound like a broken record here, but you should probably be using Mailgun, *even if*:

## You've Got The Perfect Email Regex

As nearly every article on email validation will tell you (or someone in the comments will point out), regex only gets you so far. Sure, ColdFusion 11 doesn't automatically fail long TLDs, but that's just because [it accepts anything after the @](http://www.bennadel.com/blog/2764-coldfusion-11-accepts-all-top-level-domains-tld-for-isvalid-email-validation.htm#comments_46201). Beyond that, regex won't tell you if the domain exists, or, more importantly, if the domain has MX records. Rather than continuing to point out the shortcomings of regex, let's dive into the:

## Benefits of Mailgun's Validation API

Mailgun's API reference outlines four points as the basis of their [email validation service](https://documentation.mailgun.com/api-email-validation.html). I've added some notes on each:

* **Syntax checks (RFC defined grammar)**: From what I have tested, their syntax checks match the RFC specs exactly; they get it right, including the edge cases. This isn't easy, and already puts them ahead of the other options I outlined.
* **DNS validation**: Taking it a step beyond syntax, the DNS validation confirms that the domain is valid and has MX records. RFC syntax doesn't matter, if it's <faker@totally.notreal>.
* **Spell checks**: The number of people who mistype their email addresses in forms is genuinely surprising to me; based on the websites I've worked on, it's not uncommon. One of the fields that Mailgun returns with validation requests is *did_you_mean*. So, when the user mistakenly enters <somebody@yaho.com>, the API helpfully suggests <somebody@yahoo.com>
* **Email Service Provider (ESP) specific local-part grammar (if available)**: This is yet another way that Mailgun's service goes beyond even the best regex. Here's an example: Gmail allows plus-addressing, but Yahoo and Outlook do not. So, while both [matthew+thisworks@gmail.com](mailto:matthew+thisworks@gmail.com) and [matthew+thisdoesnt@yahoo.com](mailto:matthew+thisdoesnt@yahoo.com) are syntactically valid, Mailgun knows that the second is invalid, due to Yahoo's local-part rules.

So, this might all sound good and interesting, but the goal here, of course, is a tool that we can just drop in and use. Convenience might be secondary to accuracy, but not by much. No worries; the Mailgun API is well documented and very straightforward. For the frontend, they actually provide a [jQuery plugin](https://documentation.mailgun.com/api-email-validation.html#jquery-plugin). For server-side validation, I've got a bit of code to get you started.

## Using the Mailgun API with CFML

I've started working on a CFML wrapper for the Mailgun API: <https://github.com/mjclemente/mailgun.cfc>. When it's done, it will get a blog post of its own. For now, while it's far from finished, it does include the validation method.[^5] The wrapper is easy to init, and validating an email address is straightforward:

```
//probably best done as an application variable, or a utility service in a framework
mailGun = new com.mailgun( secretApiKey = 'key-xxx', publicApiKey = 'pubkey-xxx', domain = 'yourdomain.com', baseUrl = 'https://api.mailgun.net/v3', includeRaw = false );

email = 'test@test.com';

writeDump( mailGun.validate( email ) );
```

And here's the result:

![Mailguncfc email validation dump](/public/assets/images/mailguncfc-validation-dump.png)


So, there you have it. Here's the closing TL;DR. 1) There's a lot of problems with CFML email validation. 2) Mailgun's email validation service is far more accurate than other approaches. 3) It's really easy to use. Don't reinvent the wheel; you should probably be validating email with Mailgun.[^6]

<hr />
[^1]: You can sign up for free, no credit card required, and use their sandbox server to send 300 emails a day (to 5 authorized recipients), or add your own domain and send 10,000 a month for free. In my opinion, that's a developer friendly service. You do need a mobile phone number as well.
[^2]: While multiple subsequent versions have been released, as of the [2016 State of the CF Union Survey](http://www.teratech.com/blog/index.cfm/2016/2/19/State-of-the-CF-Union-Survey-2016--Results#cfmlenginesused), a substantial number of developers are still using ColdFusion 9 and/or 10.
[^3]: [RCF 2822, section 3.2.4](https://tools.ietf.org/html/rfc2822#section-3.2.4)
[^4]: For more on this, see [Ben Nadel's post](http://www.bennadel.com/blog/2593-isvalid-accepts-emails-with-leading-and-trailing-whitespace-in-coldfusion.htm) and then try to understand how [this bug](https://bugbase.adobe.com/index.cfm?event=bug&id=3725691) was closed as "NotABug".
[^5]: Dominic Watson has an older, but more fully fleshed out Mailgun API wrapper: <https://github.com/DominicWatson/cfmailgun>. It was set up for version 2 of their API, though I think most of the methods should still work with v3. However, it does not contain the validation methods.
[^6]: Update - 09/02/2016: For those not comfortable relying on a third-party - in the comments, [@JamoCA](https://github.com/JamoCA) mentioned an [alternative approach](/2016/09/01/validating-email-with-mailgun-coldfusion.html#comment-2873168061). Check it out - it's certainly better than `isValid()` and has most of the benefits of using Mailgun.
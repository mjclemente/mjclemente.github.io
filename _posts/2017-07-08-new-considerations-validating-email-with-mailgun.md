---
date: 2017-07-08
published: true
title: Updated Considerations When Validating Email with Mailgun
layout: post
tags: [coldfusion, cfml, mailgun]
---
Well, they say that *all good things must come to an end*, which turns out to be the case for Mailgun's free email address validation API, a service [about which I've spoken highly in the past](/2016/09/01/validating-email-with-mailgun-coldfusion.html). In a June 27th [blog post](http://blog.mailgun.com/mailgun-rolls-out-changes-to-email-validation-api-including-new-pricing-model-and-features/), along with new features, they announced a usage based pricing model. So, what's changed? Should you still probably be validating your email with Mailgun? The answer, as usual, is that it depends. <!--more-->

## What's Changed?

1. **Features**: Obviously this is what they lead with in the blog post, before they discuss pricing. The first three features have to do with deeper analysis of the mailbox being verified itself.

	* **Mailbox Verification**: By default, this option is off (it adds noticeable overhead to the request time). As the API docs explain it, "If set to true, a mailbox verification check will be performed against the address." The API response will then include a `mailbox_verification` key, with a value of *true*, *false*, or *unknown*.

		*Unknown* is returned if the SMTP request could not be completed or mailbox verification is not supported on the target mailbox provider.[^1] Which providers support mailbox verification? They don't say. I ran some checks, and this is what I found:

		| Provider   | Support   |    |
		|---|---|---|
		| *gmail.com*   | Yes  | Consistently returned *true/false* .  Email+modifier@gmail.com format was also fully supported. |
		| *hotmail.com*   | Yes  |  |
		| *comcast.net*   | Yes  |  |
		| *att.net*   | Yes  |  |
		| *msn.com*   | Yes  |  |
		| *icloud.com*   | Yes  |  |
		| *aol.com*    | Yes/Partial  | Prone to timeouts, returning *unknown*  |
		| *outlook.com*   | Yes/Partial  | Prone to timeouts, returning *unknown*  |
		| *Office 365*   | Partial  | Valid email addresses returned *true*. Invalid returned *unknown*  |
		| *yahoo.com*   | No  | Consistently returned *unknown*  |

		Conclusion: It's an impressive feature, with fairly wide support, and you get an answer, most of the time.

	* **Role-based Address Check**: Effectively, this indicates whether a mailbox is a distribution list. It's returned with every validation request. I didn't have a large sample size, but I found it to be accurate. However, I don't know what specific actions I would take, based on knowing this.

	* **Disposable Mailbox Detection**: On the other hand, the value of knowing if an email address is from a known disposable mailbox provider is readily apparent. As with the role-based address check, this is returned with every verification request. Does it work? I did a quick check, after Googling "disposable email addresses" and using the first few results.

		| Provider   | Address   | Correct? |
		|---|---|---|
		| *guerrillamail.com*   | testingmailgun@sharklasers.com  | Yes |
		| *mailinator.com*   | ZippityTinkerton@mailinator.com  | Yes |
		| *getnada.com*   | zyci@cars2.club  | Yes |
		| *emailondeck.com*   | valkyrie38@ho3twwn.com  | **NO** |
		| *throwawaymail.com*   | chaprulamu@ibsats.com  | **NO** |

		Conclusion: It's another helpful feature, and you get the right answer more often than you would without it, but it's not foolproof.

	* **Rate Limiting of Validation API Key**: This final feature is not related to email verification, but rather account management. There seem to be some issues and bugs with the documentation and implementation of this feature, which I'll discuss after explaining it.

		By default, this feature is *not* enabled, so there is no limit on email validation API calls. Once enabled, it is supposed to give you the ability to set an upper limit for public API calls, after which additional requests are denied. This is important for controlling costs, but also because every existing implementation of the Mailgun email validation API is set up with a public API key and many of them are web accessible. There's obviously a heightened potential for abuse in these cases, so being able to stop excessive API calls is essential.

		So, my first question was, is the limit daily or monthly? The documentation is contradictory. The [introduction to the email validation portion of the API documentation](http://mailgun-documentation.readthedocs.io/en/latest/api-email-validation.html#email-validation) indicates that the limit is monthly, as do the email alerts.

		![Mailgun email validation API monthly limit](/public/assets/images/mailgun-email-validation-api-monthly-limit.png)

		![Mailgun email validation monthly limit alert](/public/assets/images/mailgun-email-validation-monthly-limit-alert.png)

		However, the portions of the API documentation discussing access via the private API key suggest that it is a daily limit.

		![Mailgun email validation API daily limit](/public/assets/images/mailgun-email-validation-api-daily-limit.png)

		I reached out to the Mailgun team on Twitter for clarification, but they haven't responded. Of more concern is that in my testing, I've somehow ended up above the limit that I set, and requests are not being denied.

		![Mailgun API daily limit exceeded error](/public/assets/images/mailgun-email-validation-limit-exceeded-error.png)

		I plan on reaching out to Mailgun again about this. I'll update this post if this situation is clarified or corrected.


2. **SLAs**: Honestly, a service level agreement for email validation isn't of much interest or use to me, though apparently there are clients who want it. More power to them for providing it. Of significantly more interest, as [Disqus user @chris points out](http://disq.us/p/1k6ceyu), would be an SLA for the base sending API; those outages are frustrating.

3. **Pricing**: This is the kicker when it comes to reevaluating if Mailgun is the right solution for your email validations. The first 100 validations are free each month, but after that, they start at $.01 per validation. Here's a link to the [pricing calculator](https://www.mailgun.com/pricing-validations).

	I understand the need for a company to make money to pay the bills and keep offering their services. However, I tend to side with the majority of comments on the article, the new pricing model seems a bit off. Or, perhaps better put, it's not very 'develop-friendly'.

	For smaller users, the 10,000 free messages included in the sending API is generous and affordable; it makes testing possible and strikes a good balance where the costs largely fall on users who have grown to the point where they can/should be able to handle it.

	On the other hand, the 100 validations disappear very quickly, especially when testing. While writing this post I've used 79 of mine. As a developer, it's much harder to choose Mailgun for projects I'm starting when the free cap is low and the costs jump rapidly.

## Should I Still Be Using Mailgun to Validate Emails?

The Mailgun email validation API is fantastic, and when it was free, using it was an easy decision. The new features are promising, and can provide a degree of certainty when dealing with user email addresses that I don't think is possible elsewhere. If this is compelling proposition for you, then you have your answer.

That being said, I no longer have a strong argument here; you need to decide for yourself, based on your use cases and budget. For a lot of developers, especially with smaller projects, a free service or method that's slightly inferior likely trumps a fantastic but expensive one.

<hr />
[^1]: It doesn't appear in the documentation, but it also returns `unknown` if the email address is *de facto* invalid. I would have expected false in those cases.
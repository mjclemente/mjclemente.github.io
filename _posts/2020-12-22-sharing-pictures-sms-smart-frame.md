---
published: true
title: "How Our Family Uses SMS and Smart Picture Frames to Connect During Remote Holidays"
layout: post
tags: [pipedream, nodejs, serverless, twilio, sms, sendgrid]
typora-root-url: ../../blog.mattclemente.com
---
Two days before Thanksgiving, I decided to put together a simple way for our family to share pictures, because we couldn't be together in person. The idea: smart photo frames that everyone could text pictures to. Here's how I got it working.
<!--more-->

**tldr;** A [Pipedream workflow](https://pipedream.com/@mjclemente/sms-pictures-to-email-p_wOCgyw1) parses SMS messages sent to a Twilio phone number, extracting the pictures and then using SendGrid to email them to the dedicated email addresses used by our Pix-Star photo frames.

* TOC
{:toc}
## Background

I come from a large family and while we're scattered across the country, most of us usually make it home for Thanksgiving. Not this year, thanks to COVID. 

So, I was looking for something to connect four generations of people with wildly differing degrees of technological savviness and engagement. Some use Android, others iOs; some are on social media, others aren't. And I have no idea what the cool kids are using to share photos now.[^1]

Text messages (SMS) seemed like the best candidate to bridge the platform/technology divide. But just sharing images in a group chat would be really disruptive on a holiday - phones buzzing non-stop, everyone constantly looking down at them. Digital photo frames, silently updating to include the latest pictures, appeared a promising solution, though I'd never used one before.

## The Goal

Here's the setup I was aiming for:

- Every household gets a picture frame. 
- A phone number is shared with everyone. This is the "Shared Photos" number.
- Pictures sent via text to the Shared Photos number show up on all the frames automatically.
- Everyone enjoys their day, occasionally stopping to see the new pictures that others have shared.

The problem to be solved, then, was how to get the pictures from the text messages to the frames.

While the resulting flow is far from perfect (see the [shortcomings](#problems-and-limitations) below), our family genuinely enjoyed using it, finding new pictures on the frames to be an ongoing source of delight throughout the day (and afterward). 

## Prerequisites

Here's what's needed to get this picture-sharing process up and running:

- **Wifi-enabled picture frames**:  
	It seems that wifi picture frames are still fairly novel; there are a handful of options on Amazon, with varying degrees of credibility and functionality. I didn't have a lot of time to do research, so at the [recommendation of Tom's Guide](https://www.tomsguide.com/best-picks/best-digital-photo-frames), I went with [Pix-Star photo frames](https://www.pix-star.com/),  as it was clear that I could at least email photos to them.[^2]
- **Twilio account with phone number**:  
	No surprise here; Twilio handles the SMS portion of the workflow. You'll need a paid account, so that you can [buy a number](https://support.twilio.com/hc/en-us/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console) (it seems that most cost about $1/month); don't forget to make sure that the number supports SMS and MMS. There's no need to configure it yet; that will happen automatically when we add it as a source in Pipedream.
- **SendGrid account**:  
	You could really use any API based email provider here. SendGrid has a free tier that includes up to 100 emails a day, which should be sufficient for a lot of family-image sharing purposes. 
- **Pipedream account**:  
	I've [written about Pipedream](/2020/09/06/pipedream-uptime-monitoring.html#some-pipedream-context) before. It's a developer-friendly service (with a generous free tier). In this case, Pipedream is what makes the whole picture-sharing workflow possible, because it's where we'll write the SMS-to-email translation.

## Setting Up the Workflow

### Pix-Star Frames Setup

The first thing you'll need to do is set up the picture frames. I had the them shipped to family members, who then sent me the frame serial numbers, which I needed to set up the accounts with Pix-Star.

I don't want to get too far into the weeds with the Pix-Star frame setup, but there are some important points to cover:

- Each frame gets its own account with Pix-Star, and the first step in setting up an account is picking the frame's email address. These email addresses are the key to sending emails to the frames.
- You can manage a group of frames via a multi-frame account. More info on that [here](https://pixstar.uservoice.com/knowledgebase/articles/217397-how-can-i-manage-several-pix-star-frames-from-one). It's not necessary, but it is convenient.
- In order to send pictures to all the frames, you'll need either a multi-frame account email address (example.group@mypixstar.com) or the email addresses of all the individual frames.
- The frames should be configured to accept new pictures automatically and their slideshows switched to "Play the Inbox Photos" (that is, the ones they receive via email). These updates can be made via the web and don't require access to the frames.[^3]

### Twilio Phone number

Twilio provides a [guide for buying phone numbers](https://support.twilio.com/hc/en-us/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console), so I won't retype those instructions here. 

I found it helpful to create a contact in my phone associated with the Twilio number for this project (I even added a picture). Having a contact makes it easier to send pictures to the number from your phone's camera roll, as well as share the number/contact with family members.

### Pipedream Workflow

Here's the [Pipedream workflow for converting Twilio SMS pictures to SendGrid email attachments](https://pipedream.com/@mjclemente/sms-pictures-to-email-p_wOCgyw1). If you're logged into Pipedream, you should see an option to copy this into your account. Once copied into your account, there are a few more steps required before you can actually use the workflow.

First, you'll first need to connect your Twilio account. Pipedream provides instructions regarding what information is required. Note that you'll only need a standard API key. 

Once you've linked Twilio, you'll need to select the number from your account that you want to use as the workflow trigger. That is, when an SMS is received at this number, the workflow will be run, with the message content available as input. To complete the trigger, you'll also need to enter your Twilio Auth Token; the "SMS Response Message" field can be left blank. Finally, give it a name (like "Family Incoming SMS") and click Create Source. This takes a few seconds to complete, 

You're not quite done with this step yet; the trigger is created, but you also need to enable it. This is done by toggling the On/Off button on the top right of the trigger card. Once enabled, the workflow is ready to run for new events (SMS messages).

You'll then need to connect SendGrid. This is a matter of [creating a new API key](https://sendgrid.com/docs/ui/account-and-settings/api-keys/#creating-an-api-key) to allow Pipedream access to use your SendGrid account. This API key only needs Restricted Access (full access to "Mail Send"). 

Finally, with SendGrid connected, you just need to fill in the parameters required to send the email via SendGrid:

- **To email**: Either your multi-frame account email address or a comma separated list of the email addresses of all the individual frames.
- **Subject**: Up to you. I'm currently using "New Picture from \{\{steps.trigger.event.From}}". This is being sent to the frame email address, so it's not visible.
- **From email**: An email address that you've verified with SendGrid (or from a domain that you've verified).[^4]
- **Type**: Just set it to "text/plain".
- **Value**: This is the body of the email, and as with the Subject, it isn't really being used, but it is required. I set it to "More pictures have been shared by {\{steps.trigger.event.From}}".

You can then Save the workflow, and then Deploy it. You're ready to give it a test! Text a picture to the Twilio/Shared Photos number that you set up and it should, in a matter of minutes, appear on all the photo frames. Honestly, it was a bit of a rush the first time it worked for me.

## Problems and Limitations

This flow for handling the SMS picture-to-frame was put together quickly, and has a handful of obvious shortcomings. While some are fixable, with a bit more work, others are just inherent in the process:

- **Frames are expensive**. No real way around this. You might try to cut costs a little with repurposed tablets, but I didn't have enough time to explore that route.

- **Frame software problems**. As mentioned above, we used Pix-Star photo frames. And while the process worked, there were two issues worth noting. 

  - First, we found that an occasional photo would not make it to all the frames. I reached out to the Pix-Star support, who were very nice, but had not idea why this was happening and were not able to resolve it. Definitely a source of frustration. 
  - Second, the Pix-Star interface was less than ideal, which made it difficult to manage the frames/pictures from the web. Along with being a bit slow and clunky, there was no way to make sure that the frame and web stayed in sync - a photo could be deleted on the web, but remain on the frame. 

  I'm not sure how the experience would be different with other frame providers.

- **Picture quality is reduced.** Because we're sending MMS, network carriers limit the size of the message/files. So, a 3MB photo from your phone might end up compressed to 800KB by the carrier. Consequently, it's best to only send one picture at a time.[^5]

  Now, I should note that we didn't find this to be a problem. While lower quality than the originals, the resulting images were good enough for the size they were displayed at, especially in collage-mode.

- **Workflow doesn't delete the images from Twilio.** I haven't gotten around to including this step in the workflow. [It's possible](https://www.twilio.com/docs/sms/tutorials/how-to-receive-and-download-images-incoming-mms-node#delete-media-from-twilio), but I was rushed, and this seemed the least important thing to implement. Just keep it in mind. I believe they're automatically deleted after 13 months, and there are also tools that use the Twilio API to automate the process.

## Wrapping it up

Not much else to say here, except that our family enjoyed this so much that we've continued to use the SMS-to-frame sharing, even after Thanksgiving. We've kept the frames up, and while we're not sending pictures as frequently as we did on the holiday, sharing little moments of daily life is still delightful.



_____

[^1]: Is Snapchat still cool? Is it still around?
[^2]: It seems that [Nixplay](https://www.nixplay.com/) also provides this functionality, and *may* be a better choice. More about this in the [Problems and Limitations](#problems-and-limitations) section.
[^3]: Minor annoyance; the frames come with stock photos that need to be manually deleted.
[^4]: Another helpful tip: Within Pix-Star, you're able to restrict the email addresses that are allowed to send pictures to the frames. So it's possible to configure your frames to only accept from this email address that you're sending from here.
[^5]: The workaround here, if you need to send more pictures, or require higher quality, is to email the images directly.
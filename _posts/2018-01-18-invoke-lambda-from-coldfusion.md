---
date: 2018-01-18
published: true
title: Invoke Lambda from ColdFusion
layout: post
tags: [coldfusion, aws, lambda, lambda.cfc]
---
At Adobe's ColdFusion Summit (2017), I was able to attend [Brian Klaas's](https://github.com/brianklaas) presentation, "Level Up Your Web Apps With Amazon Web Services." Brian is an engaging speaker - he conveyed excitement, knowledge, and real-world insights into the often overwhelming AWS ecosystem. I was particularly intrigued when he explained that ColdFusion could interact directly with AWS, via the Java SDK.
<!--more-->

## The backstory

Our first attempt to use [AWS Lambda](https://aws.amazon.com/lambda/) at work, while ultimately successful, was not without frustration. Most of the difficulties arose due to our use of Amazon's [API Gateway](https://aws.amazon.com/api-gateway/) as the trigger for our Lambda Function. Working with two AWS services meant that, along with an increased cost, there was more to learn, configure, and debug - largely undermining the promise of simply calling a serverless function.

However, Brian demonstrated that we could, in fact, just *invoke Lambda from ColdFusion*; there was no need to use a second service as a trigger. The sample code from his presentation is [available on Github](https://github.com/brianklaas/awsPlaybox). I started there, with the goal of putting together a CFML component for directly invoking Lambda functions. Fortunately, Brian had already done the hardest part - setting up the objects and process necessary to use the Java SDK.

## lambda.cfc - A Component for Invoking AWS Lambda

[Lambda.cfc](https://github.com/mjclemente/lambda.cfc) is effectively a wrapper for the AWS SDK, abstracting away its complexity, and providing a simple interface for invoking Lambda functions. I believe that I documented the component fairly well in the [Github repository](https://github.com/mjclemente/lambda.cfc), so head over there if you're interested in the details of setup and configuration.

Here's a basic example of how it works:

```cfc
//init the component with valid creds
lambda = new path.to.lambda( accessKey = xxx, secretKey = xxx );

//ARN (Amazon Resource Name) of the Lambda function you're invoking
arn = 'arn:aws:lambda:us-east-1:123456789098:function:yourArn';

//set up any arguments the function takes as a payload
payload = {
	"arg1" : value,
	"arg2" : value2
};

//pass them in, and you've got your result
result = lambda.invokeFunction( arn, payload );
```

That's about all there is to it. The only real variation is that you have the option to [set a default Lambda ARN](https://github.com/mjclemente/lambda.cfc#setting-a-default-lambda-function). This can be convenient if your component is only meant to invoke one Lambda function, as it saves you from having to pass in the ARN parameter every time. The `defaultArn` argument can be provided when the component is initialized, or set later, as in this example:

```cfc
///set the default ARN on an existing component
lambda.setDefaultArn( arn );

//invokes the Lambda function specified by the default ARN
result = lambda.invoke( payload );
```

Hopefully other developers find this as useful as we have. If you have questions or issues with it, I'll do my best to help. And if you want to dive further into using AWS Services with ColdFusion, again, check out Brian's [AWS Playbox](https://github.com/brianklaas/awsPlaybox) or, if you have the opportunity, go see him speak.

---
date: 2018-01-11
published: true
title: Parsing AWS ARNs with CFML
layout: post
tags: [coldfusion, aws, lambda]
---
I've recently begun working on a CFML project that involves interacting with [AWS Lambda](https://aws.amazon.com/lambda/). I'll be posting more on that later, but one of the helpful bits of code that came out of the project was a small function to parse Amazon Resource Names (ARNs) into their component parts.
<!--more-->

Why would you want to parse an ARN? Because they contain a lot of information - including the AWS service, region, account, and resource identifier. Extracting that information from the ARN means you don't need to input it separately.

Amazon has fairly extensive [documentation of the ARN format](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html); the general structure is consistent and straightforward:

```
arn:partition:service:region:account-id:resource
arn:partition:service:region:account-id:resourcetype/resource
arn:partition:service:region:account-id:resourcetype:resource
```
Some searching brought up parsers for [Node](https://www.npmjs.com/package/aws-arn-parser), [Go](https://github.com/gigawattio/awsarn), [Clojure](https://github.com/skybet/aws-arn), and [Python](https://gist.github.com/gene1wood/5299969edc4ef21d8efcfea52158dd40). I looked at them all, but primarily based mine on the Python script. You can see it below; it's also available [as a Gist](https://gist.github.com/mjclemente/37d8ccd25b3ae5f0a31f9b209b0a8a74).

```cfc
/**
* @hint Parses an Amazon Resource Name (ARN) and returns its component parts as an object.
* This follows the general format of ARNs outlined by Amazon (http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html), but does not fully account for all possible formats
* Derived from https://gist.github.com/gene1wood/5299969edc4ef21d8efcfea52158dd40
*/
public struct function parseArn( required string arn ) {
  var elements = arn.listToArray( ':', true );
  var result = {
    'original' : arn,
    'arn' : elements[1],
    'partition' : elements[2],
    'service' : elements[3],
    'region' : elements[4],
    'account' : elements[5]
  };

  if ( elements.len() >= 7 ) {
    result[ 'resourcetype' ] = elements[6];
    result[ 'resource' ] = elements[7];
  } else if ( !elements[6].find( '/' ) ) {
    result[ 'resource' ] = elements[6];
    result[ 'resourcetype' ] = '';
  } else {
    result[ 'resourcetype' ] = elements[6].listFirst( '/' );
    result[ 'resource' ] = elements[6].listRest( '/' );
  }

  return result;
}
```

If you pass in a basic Lambda ARN, like this:

```cfc
arn = 'arn:aws:lambda:us-east-1:123456789012:function:ProcessKinesisRecords';
result = parseArn( arn );
writeDump( var='#result#' );
```
You should get back a struct with the resource information:

![parse aws arn result struct](/public/assets/images/parse-aws-arn-result-struct.png)

A few notes in closing:

* AWS has a staggering range of services, some of which deviate from the standard ARN format. Bottom line, this won't work for every ARN; it's a basic parser.
* You'll need to remove the `var` scoping if you're not using this within a CFC... but you should probably be using it in a CFC.
* It uses member functions, so you won't be able to use it on ACF 10, though the changes to backport it should be trivial.


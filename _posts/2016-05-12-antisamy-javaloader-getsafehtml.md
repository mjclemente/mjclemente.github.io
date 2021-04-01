---
published: true
title: Using AntiSamy with ColdFusion 11
layout: post
tags: [coldfusion,antisamy,javaloader]
---
We recently had the need to validate rich text input in one of our applications. In the past we've rolled our own validation, using various regex strings. While this worked, we're always looking for better ways to solve problems, which is why I was intrigued to find the [isSafeHTML](https://cfdocs.org/issafehtml) and [getSafeHTML](https://cfdocs.org/getsafehtml) functions. These were added in ColdFusion 11, but I had not heard of them. While we ultimately didn't end up using them, that's how I fell down this particular rabbit hole.
<!--more-->

The first issue we ran across was that the functions haven't been added to Lucee yet. Granted, we're not on Lucee, but we're exploring it, so I was hesitant to settle on a solution that would make that transition more difficult.[^1] As indicated in the documentation and [various](http://cfpavankumar.blogspot.com/2014/04/using-antisamy-framework-with.html) [references](https://www.raymondcamden.com/2014/04/09/getSafeHTML-and-ColdFusion-11/) to the functions, they make use of the [AntiSamy Project](https://www.owasp.org/index.php/Category:OWASP_AntiSamy_Project).[^2]  

This ultimately led me to Pete Freitag's post about [Using AntiSamy with ColdFusion](http://www.petefreitag.com/item/760.cfm). It's a guide  from the CF8/CF9 days, and walks you through implementing the AntiSamy project using the java library directly. We settled on using this general approach, instead of the built in ColdFusion functions for three reasons:

* We wanted to maintain compatibility with Lucee.
* ColdFusion's implementation [does not use the most recent AntiSamy jars](http://stackoverflow.com/questions/32428811/how-to-update-the-antisamy-jar-file-in-coldfusion-11) (1.4.4 vs 1.5.3).
* Using the project jars directly enables us to return an array of error messages when input fails validation, explaining what failed and why. As they say on the AntiSamy project page, "rejecting [user] input without any clue as to why is jolting and annoying". This information isn't available with the ColdFusion functions.  

I first thought that we'd be able to utilize ColdFusion's built in java library loading (available since CF10). This approach is explained [here](http://coldfusion-tip.blogspot.com/2013/12/coldfusion-antisamy-library-integration.html), but I found that when using that approach, I would end up with the AntiSamy jar that ships with ColdFusion. I mentioned this on the CFML Slack channel, and Pete Freitag explained: 

> if using `this.javaSettings` I think it will always load the CF version of the jar, you can try `this.javaSettings.loadColdFusionClassPath=false` but I think it will still use the CF one... I think the only way to do it is to create a custom classloader (which is what javaloader does)

So, our approach necessitated using Mark Mandel's [Javaloader](https://github.com/markmandel/JavaLoader) project, which, while old, [still gets a lot of love](http://www.bennadel.com/blog/2943-using-launchdarkly-with-coldfusion-and-javaloader.htm), so it seemed worth exploring. The only complication with using the external javaloader, is that [it can cause memory issues](https://github.com/markmandel/JavaLoader/wiki/Memory-Consumption) if not stored in the server scope. Jamie Krug built a wrapper function, [JavaLoaderFactory](https://github.com/jamiekrug/JavaLoaderFactory) to abstract/simplify this process. It takes the arguments you supply to create a JavaLoader instance and uses them to create a unique identifier. If that identifier/instance already exists in the server scope, it gets returned, and if not, it gets created. As I understand it, by only creating/using the instances that you need, the memory leak issue is averted, because they don't need to be garbage collected.

To use JavaLoader with the factory wrapper, you need to make sure it's accessible to your application. You can do this by including it in your application, or by adding a mapping. For example, if you were to include it a directory above your site root, you could add this to application.cfc:

	this.mappings['/javaloader'] = GetDirectoryFromPath( ExpandPath( "../javaloader/") );

You then need to add the `javaloaderfactory.cfc` wrapper to your project. We're using FW/1, so we just dropped it in our services folder. If you're not using a framework, just put it wherever your store your cfcs, so that you can create an instance of it:

	factoryLoader = createObject( "component", "path.to.javaloaderfactory" ).init();
	
With the JavaLoader and factory ready, the next step is to set up your arguments; this is the array of jars you need loaded in order to create the AntiSamy instance. We downloaded the most recent jars and dependancies from the [AntiSamy Maven repository](http://mvnrepository.com/artifact/org.owasp.antisamy/antisamy/1.5.3) and included them in a `/lib/` folder. It doesn't really matter where you put them. Once you've got them, you can create an instance:
	
	//set up array of jars
	jarArray = [
      	expandPath('/lib/antisamy/antisamy-1.5.3.jar'),
      	expandPath('/lib/antisamy/batik-css-1.7.jar'),
      	expandPath('/lib/antisamy/xml-apis-ext-1.3.04.jar'),
      	expandPath('/lib/antisamy/batik-util-1.7.jar'),
      	expandPath('/lib/antisamy/batik-ext-1.7.jar'),
      	expandPath('/lib/antisamy/nekohtml-1.9.16.jar'),
      	expandPath('/lib/antisamy/xercesImpl-2.9.1.jar')
	];
	
	//use the factory to get an instance of the JavaLoader
	javaloader = factoryLoader.getJavaLoader( jarArray );
	
	//use the JavaLoader to create an AntiSamy instance
	antiSamy = javaLoader.create( 'org.owasp.validator.html.AntiSamy').init();
	
We do this in the `init()` of a validation service, and store the AntiSamy instance in the variables scope, so that we can have it available wherever we're doing validation. Again, without a framework[^3], you might want to store it in the application scope.

You're almost, but not entirely done at this point. AntiSamy is based on xml configuration files that you need to supply when validating. You can find samples on the [Google Code Archive for the project](https://code.google.com/archive/p/owaspantisamy/downloads). The [Developer Guide](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/owaspantisamy/Developer%20Guide.pdf) listed there is fairly helpful in explaining the configuration of the xml, and the 5 example policies are excellent starting points. So, add your policy to your application somewhere (or add more than 1, if you have different requirements for different types of input). Then, finally, you can validate/clean text. Here are two examples: 
	
Example 1: Returning an array of errors:	

	//take your untrusted input
	unTrustedInput = '<p>Hi! <IMG SRC="javascript:alert('XSS');"></p>';
	
	errors = [];
	//define your policy file
	policyFile = expandPath( '/path/to/antisamy/config.xml' );
	
	//pass the untrusted input and policy file into your AntiSamy instance
	errors = antiSamy.scan( unTrustedInput, policyFile ).getErrorMessages();
	writeDump(errors);

Example 2: Returning cleaned data:

	//take your untrusted input
	unTrustedInput = '<p>Hi! <IMG SRC="javascript:alert('XSS');"></p>';
	
	trustedInput = '';
	//define your policy file
	policyFile = expandPath( '/path/to/antisamy/config.xml' );
	
	//pass the untrusted input and policy file into your AntiSamy instance
	trustedInput = antiSamy.scan( unTrustedInput, policyFile ).getCleanHTML();
	writeDump(trustedInput);

We were thrilled with the result. The level of control provided by the config files was excellent, the result was CFML engine independent, and the information returned was useful. As Pete Freitag did in his original post, I thought it would be helpful to include a full working version. You can see/get it here: [https://github.com/mjclemente/antisamywithcoldfusion](https://github.com/mjclemente/antisamywithcoldfusion). 


<hr />
[^1]: I added a ticket regarding the functions for Lucee, feel free to upvote or comment: [https://luceeserver.atlassian.net/browse/LDEV-838](https://luceeserver.atlassian.net/browse/LDEV-838)

[^2]: The project is inactive, which bothered me a little. However, it's still being used within the active [ESAPI Project](https://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API), and I wasn't able to find much about a successor. There are a few out there, but AntiSamy still seems to still be an acceptable solution.

[^3]: Seriously, use a framework. Use FW/1. It makes life so much easier.
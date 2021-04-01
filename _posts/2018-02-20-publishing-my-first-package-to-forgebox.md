---
published: true
title: Publishing My First Package to ForgeBox
layout: post
tags: [coldfusion, cfml, forgebox]
---
[ForgeBox.io](https://www.forgebox.io/), in case you didn't know, is directory of CFML packages - bits of code both large and small - written and shared by the developer community to make all of our lives easier. I've written a number of open source projects hosted on Github, so I figured it was time that I began adding them to ForgeBox. For posterity (or just my own reference), here's the process.
<!--more-->

Let's define what we're talking about first. What is a package? The [CommandBox docs](https://commandbox.ortusbooks.com/content/packages/creating_packages/creating_packages.html) explain it succinctly:

> Packages are quite simply a folder that contains some code and a box.json file. A package can be a simple CFC, a self-contained library, or even an entire application.

So, `folder + box.json = package`. The actual code in the package folder might be as simple as a file with a single method for [comparing string similarity](https://www.forgebox.io/view/String-Similarity), as complex as [an entire CMS](https://www.forgebox.io/view/contentbox-stable-updates), or something in-between. The `box.json`, located in the root folder of the package, contains a description of the package contents and additional meta-data.

## The ForgeBox Package Creation Process

On with the process. I'm going to assume that you already have [CommandBox installed](https://www.ortussolutions.com/products/commandbox); if not, you should. In my case, these projects already exist on GitHub.

1. The first step is to set up a ForgeBox account. You're gonna need it to manage your packages: [https://www.forgebox.io/security/registration](https://www.forgebox.io/security/registration)

2. Hop into the terminal and navigate to the project you're going to add. Run `box` within the project folder, to start CommandBox.

3. Now I'm going to assume that you haven't already authenticated your user with ForgeBox, but if you want to check, run `forgebox whoami`. If you're not registered, you'll get the message: "You don't have a Forgebox API token set."

4. Log in to ForgeBox from the commandline:

	```text
	forgebox login username="yourUsername" password="yourPassword"
	```
	You'll get the message: "User [yourUsername] authenticated successfully with [forgebox]"
5. Now it's time to put together the actual package properties that will generate a `box.json` and create the first version of the package.[^1] There are a more than [two dozen possible properties](https://commandbox.ortusbooks.com/content/packages/boxjson/boxjson.html#sample-boxjson). I'm not going to repeat the docs verbatim, or go over every property, but here are some points that I found helpful regarding some of the common properties:
	* **Slug**: This is what people will use when installing your package from ForgeBox. Spaces and special characters are not allowed. You can use `forgebox slugcheck` to check if a slug is available.
	* **Type**: This area was a bit hazy for me; the documentation on package types is a work in progress. Here's what I learned:
		* Packages are organized on ForgeBox by *Type*.
		* To view a list of available package types, run `forgebox types`... or just go to the website.
		*  The package type is more than simply meta-data; it can also determine how the package is installed. Of particular note for me was that CommandBox tries to determine the proper installation directory based on the package type. For example, `modules`, `plugins`, and `interceptors` are automatically installed in a directory of the same name. If the package type is `jars`, the package is installed in `/lib`.
		*  Framework/application specific package types, such as those for Preside CMS, CommandBox, etc. may have their own conventions applied to the installation.
		* The current working directory is used for the installation if no conventions are found for the package type.

		All of that being said, I opted for `modules`, as that seemed to be the most common type used for API wrapper type packages.
	* **Directory**: If you want to override the installation directory conventions outlined above, based on the package type, you can do that here. This specifies a directory, relative to the webroot, in which the package will be installed.
	* **Version**: [Straightforward semantic versioning](https://commandbox.ortusbooks.com/content/packages/managing_version.html) - if you don't include a version, it defaults to 0.0.0. Because I already version my GitHub projects, I'll do my best to keep the ForgeBox and GitHub versions in sync. However, it's worth noting that this seems to be a pain-point for package maintainers, based on the ForgeBox packages I've tried to install in the past. Frequently I've seen versions get bumped on one platform but not on the other, resulting in installation difficulties.
	* **Location**: This is the actual download location of the package. There are two ways to set this:
		* Directly link to a zip that contains the package: `https://github.com/mjclemente/sendgrid.cfc/archive/v0.5.5.zip`
		* Use your GitHub username, the name of the repo, and the version: `mjclemente/sendgrid.cfc#v0.5.5`

		It's important to be aware that, in order to version your package, you *need to update the location to point to the correct version*. This is another point of friction for maintainers that I've noticed; forgetting to update the location link to point to the correct version is an easy mistake.

		If you're not versioning your package or repo, the options above would be `https://github.com/mjclemente/sendgrid.cfc/archive/master.zip` or `mjclemente/sendgrid.cfc`, respectively.

	Okay, we understand the properties, more or less. On to package initialization!

6. But wait! There's an easier way to get started with the package. The good folks at Ortus provide a wizard for constructing your basic package. In your project root, just run `package init --wizard`. It will walk you through entering the basic properties. Here's what mine looked like:
	![CommandBox package wizard](/public/assets/images/commandbox-package-wizard.png)
	That... was really, really easy. I was very impressed.

7. But don't go trying to publish this yet; it would get rejected. We still need to set both a `location` and `type`. Here's what I ran; you would need to update this with your specific values:

	```text
	package set location=mjclemente/sendgrid.cfc#v0.5.5
	package set type=modules
	```
8. And now, we're ready to publish it: `forgebox publish`
	![ForgeBox publish success](/public/assets/images/forgebox-publish-success.png)

## The Package is Published

That's it! My first package is live on ForgeBox. You can see it here: [SendGrid.cfc](https://www.forgebox.io/view/sendgridcfc). And, more importantly, you can install it using CommandBox: `install sendgridcfc`

![ForgeBox install SendGrid.cfc](/public/assets/images/forgebox-install-sendgridcfc-package.png)

A final note; there are a number of other properties that can be set for the package, regarding its changelog, issue tracking, its license, etc. I plan on filling mine out more completely, as these are certainly valuable for developers attempting to use your project.

And, on to more package creation and publishing!


<hr>
[^1]: Here are the docs on [Basic Package data](https://commandbox.ortusbooks.com/content/packages/boxjson/basic_package_data.html) and [Extended Package data](https://commandbox.ortusbooks.com/content/packages/boxjson/extended_package_data.html), which are probably more helpful than what I'm outlining.
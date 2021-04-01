---
published: true
title: Building My First ColdBox Modules
layout: post
tags: [coldfusion, cfml, forgebox, coldbox]
---
Ok, the title isn't entirely accurate. The process discussed here isn't so much "building" a module as "adding ColdBox functionality" to an existing ForgeBox package - *boxifying* it, one might say. I've never built a ColdBox application, but I've been increasingly interested in the framework, so this was a helpful and straightfoward first step toward better understanding it.

<!--more-->
This was supposed to be a short post. I'm sorry; it's not.

* TOC
{:toc}

## Getting Started
No interest in ColdBox? Check out my [note at the end](#a-note-to-non-coldbox-developers); these projects aren't leaving you behind!

In an earlier post, I wrote about [publishing a package to ForgeBox](/2018/02/20/publishing-my-first-package-to-forgebox.html); that's more or less where this picks up. I received a PR from [Matt Gifford](https://github.com/coldfumonkeh), turning [sendgrid.cfc](https://github.com/mjclemente/sendgrid.cfc), my first ForgeBox package, into a ColdBox module. This conveniently occurred while I was attending [Into The Box](/2018/04/26/into-the-box-2018.html), so I was able to ask [Eric Peterson](https://github.com/elpete) from Ortus all my questions about the "boxification" process. Let's dive in with the basics.

## What's a Module?
A module is a reusable package of functionality that you can add to your application. You might be asking yourself, how is a module different from a package? In most cases, it isn't worth making a distinction, and the terms can be used synonomously.[^1] Modules might provide string parsing, interaction with an API, or security features. Browsing [modules on ForgeBox](https://www.forgebox.io/type/modules) shows the range of functionality they can add.

Here, it's probably helpful to outline the various possible relationships between modules and the frameworks using them:

* __Framework-independent__: Many modules on ForgeBox, like the first ones I published, don't need a specific framework (or any framework), in order to function. This doesn't mean that they're easy to use within any framework - only that their functionality isn't dependant on a framework. Because they don't follow the conventions of ColdBox, they're not particularly easy to leverage within the framework.
* __Framework-dependent__: In CFML, this almost universally means ColdBox-dependent. These modules are built relying on the conventions, dependency wiring, etc. provided by ColdBox. By leveraging the framework, they are able to provide functionality that would be difficult, if not impossible to replicate using the framework-independent approach. Additionally, the functionality these modules provide may only make sense within a ColdBox application. Using them outside ColdBox is not possible.
* __Framework-agnostic__: These modules are able to work within any framework, or no framework. Adding ColdBox support to framework-independent modules results in functionality that the entire CFML community can leverage.

Not every module can (or should) be framework-agnostic. Still, in writing this I've come to realize that many modules in those first two categories could be in the third, but it requires a bit more work from developers (myself included). While we can work toward removing unnecessary coupling from some ColdBox modules, the area with the most room for growth, I believe, is in adding ColdBox support to existing, framework-independent modules.

To that end, let's look at what it takes to _boxify_ an existing module.

## So What's a ColdBox Module?
Maybe, after a bit more experience with the framework, I'll be able to provide a more complete guide to ColdBox modules.[^2] For our purposes, here are some basics:

* The ColdBox convention is to store modules in the `/modules` folder (that's where they're installed when you run `box install module-name`)
* Their settings are configured in `/config/ColdBox.cfc` within the `moduleSettings` struct.[^3]
* When a ColdBox app starts, the modules are automatically loaded and available to use.[^4]
* At it's most basic, the only requirement for a ColdBox module is that it must have a `ModuleConfig.cfc` in the root of the folder, containing a `configure()` method.

__The `ModuleConfig.cfc` is the key here. It tells the framework how to load, unload, and use the module and it's the only file you need to add to non-ColdBox modules in order to provide framework support.__ Let's take a closer look at how to set up a `ModuleConfig.cfc`.

## Adding ColdBox Support: `ModuleConfig.cfc`

While there's a lot you can do with `ModuleConfig.cfc`, the basics are pretty straightforward. I'll be using the [configuration](https://github.com/mjclemente/sendgrid.cfc/blob/v0.7.0/ModuleConfig.cfc) I ultimately settled on for SendGrid.cfc for examples.

For simple modules, the config file can be broken down into three sections: [properties](#properties), [configuration](#configuration), and [loading](#loading).

### Properties
At the start of your `ModuleConfig.cfc` you can add a more than a dozen public properties. They are all optional; some are descriptive, others adjust how the module interacts with the ColdBox application.

The descriptive properties include `name`, `author`, `webURL`, `description`, and `version`. It's worth noting that these existed prior to ForgeBox so their utility has largely been replaced by `box.json`. I ended up including them all except for `version`, but the truth is the importance of the descriptive properties is negligible. Include them if you like, but feel free to disregard some or all of them.

The remaining possible properties govern how your module interacts with the larger application. If you're porting a framework-independent module to ColdBox, you should be fine ignoring these; the defaults are fine. For more information about the module properties, [check the docs](https://coldbox.ortusbooks.com/hmvc/modules/moduleconfig/public-module-properties-directives). Here are the properties in the `ModuleConfig.cfc` of SendGrid.cfc:

```
this.title = "SendGrid Web API v3";
this.author = "Matthew J. Clemente";
this.webURL = "https://github.com/mjclemente/sendgrid.cfc";
this.description = "A wrapper for the SendGrid Web API v3";
```

### Configuration
As mentioned previously, each module's `ModuleConfig.cfc` must contain a `configure()` method; if it doesn't, the module will be ignored by the framework.

On startup, ColdBox creates a struct for storing settings; within that struct is a key named `modules` containing the configuration for each module in the app. This information comes from the `configure()` method.[^5]

There are a number of module-specific properties and behaviors that you can customize within `configure()`, but you don't need to learn them now. In fact, unless you're building a module with more complex framework interactions, __you only need to deal with one property here: `settings`__.

It's very straightforward: within `configure()`, use the `settings` struct to provide the default settings for your module. Here's the SendGrid.cfc example:

```
function configure(){
  settings = {
    apiKey = '',
    baseUrl = 'https://api.sendgrid.com/v3',
    forceTestMode = false
  };
}
```
Developers using the module can then override these values from within their `/config/ColdBox.cfc`, like so:
```
moduleSettings = {
  sendgridcfc = {
    apiKey = 'xxxxxxxxxx',
    forceTestMode = true
  }
};
```
ColdBox will merge the structs, giving precedence to the custom settings. Those settings will then be available throughout the application. Which is good, because we need them in this last portion of configuring our module.

### Loading
The final step is to add the module's relevant CFCs to WireBox (the dependency injection framework), so that they can be used throughout the application. This is done within the `ModuleConfig.cfc`'s lifecycle method `onLoad()`. I'm going to provide the SendGrid.cfc example, and then outline what it's doing:
```
function onLoad(){
  binder.map( "sendgrid@sendgridcfc" )
    .to( "#moduleMapping#.sendgrid" )
    .asSingleton()
    .initWith(
      apiKey = settings.apiKey,
      baseUrl = settings.baseUrl,
      forceTestMode = settings.forceTestMode
    );
  binder.mapDirectory(
    packagePath = "#moduleMapping#/helpers",
    namespace = "@sendgridcfc"
  );
}
```
We're using `binder` here to provide WireBox with the details of how this module should be handled. Let's tackle each element of the statement:

#### `map()`

This is used to assign an identifer to our component. The convention, as I understand it, is to use `cfc@folder` for modules, so for `sendgridcfc/sendgrid.cfc` we get `sendgrid@sendgridcfc`. Throughout the application, developers can now use this identifier in property declarations wherever they need the component:
```
property name="sendgrid" inject="sendgrid@sendgridcfc";
```

#### `to()`
The next element, `to()`, explicitly tells Wirebox where the actual file is. We're using the variable `moduleMapping`, which ColdBox provides automatically; it's the path needed to create CFCs within this module (in this case it would be `/modules/sendgridcfc`).

#### `asSingleton()`
Fairly self-explanatory; we only want one instance of this CFC created; we want the single instance to persist throughout the life of the application. If we didn't provide this, WireBox would consider it a transient object and would create a new instance every time it was needed.

#### `initWith()`
This method tells WireBox the arguments it should pass into the component's `init()` method; it's also where we use the settings that we configured earlier. The merged default and application-specific settings for the module are available in `onLoad()` within a struct named `settings`, so they can be referenced as argument values.

#### `mapDirectory()`
This second use of the `binder` is because the SendGrid module has a handful of helper components located in the `sendgridcfc/helpers` directory. Instead of needing to map each individually, ColdBox provides `mapDirectory()`. With the parameter `packagePath`, we provide the directory being mapped; the `namespace` parameter provides the convention for naming CFCs within the directory. The result, in our case, is `filename@sendgridcfc`. It should be noted that, because we don't specify otherwise, these mapped CFCs are all transients.

## Putting It All Together
To recap, we've added a `ModuleConfig.cfc` to the root of our project, used its `configure()` method to provide default `settings`, and then used `onLoad()` to add the relevant CFCs to WireBox.

If you're not a ColdBox developer, you'll want a way to know that your updates have worked; I recommend using CommandBox to set up a basic ColdBox app. There's a [60 Minute Quick Start](https://coldbox.ortusbooks.com/for-newbies/60-minute-quick-start) that I found really helpful; for a 30 Second Quick Start, you can copy and paste the following:

```bash
mkdir myTestApp --cd &&
coldbox create app name=myTestApp skeleton=Simple &&
coldbox create handler name="hello" actions="index" &&
server start --rewritesEnable
```

That's it; you've got a ColdBox app up and running. Add the module you're developing to the `/modules` folder to get started.

During the development, you'll want to include the following in the `configure()` method of your app's `/config/ColdBox.cfc` to ensure the CFCs are reloaded with each request:

```
wirebox = {
  singletonReload = true
};
```

Finally, to confirm that your module is working as expected, try using it within the `hello.cfc` handler:

```
component {

  property name="sendgrid" inject="sendgrid@sendgridcfc";

  function index( event, rc, prc ){

    result = sendgrid.doSomething();
    writeDump( var='#result#', abort='true' );

  }
}
```
There should be a link to this handler, `/hello`, from the index of your test ColdBox app.

## Conclusion
That was so many words. I'm sorry. If you've got any questions please let me know! Good luck with adding ColdBox support to your modules!

___
## A Note to Non-ColdBox Developers
ColdFusion projects that support ColdBox can appear overly complex. When you're looking for straightforward code to accomplish some task, a repository filled with `.json`, `.yml`, and all other manner of config files and folders may seem like more trouble than its worth. I encourage you not to be frustrated; there's a good chance you can still use it:

* *Supporting ColdBox does not necessarily mean that ColdBox is a requirement*, so read the documentation. If ColdBox isn't required, those additional files and folders exist to provide extra conveniences and features when used within a ColdBox application, but you're fine ignoring them and just using the relevant CFC directly.
* *CommandBox is not ColdBox*. Just because a project says to "install with CommandBox" 1) doesn't mean that it needs ColdBox and 2) doesn't mean that CommandBox is the *only* way to install it.

Long story short, there is a concerted effort to make cross-framework modules that the entire CFML community can use. Furthermore, what looks like complexity to you might be there to simplify the life of a developer using a different framework or tooling. If you're having trouble using some project, consider asking for help; I've found most project maintainers will go out of their way to provide assistence if you ask nicely.

___
[^1]: Strictly speaking, here's how I understand it. Within CFML, *package* would be the more general term; anything with a `box.json` is a package. *Modules* are a subset of packages, meant to provide specific functionality. That is, every module is a package, but not every package is a module.
[^2]: For example, ColdBox modules are, in effect, mini-MVC applications. That is, they can have models, views, controllers, etc. When I first heard this, it somehow blew my mind and "just made sense", at the same time. I really want to explore the flexibility and functionality this modular architecture provides.
[^3]: See the [docs for module settings](https://coldbox.ortusbooks.com/hmvc/modules/moduleconfig/module-settings#overriding-module-settings){:target="_blank"}
[^4]: See the [module lifecycle docs](https://coldbox.ortusbooks.com/hmvc/modules/module-service/module-lifecycle){:target="_blank"}
[^5]: See the [docs for the `configure()` method](https://coldbox.ortusbooks.com/hmvc/modules/moduleconfig/the-configure-method)

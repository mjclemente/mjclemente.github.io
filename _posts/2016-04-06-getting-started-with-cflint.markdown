---
published: true
title: Getting Started with CFLint
layout: post
tags: [coldfusion,cfml,cflint]
---
We recently encountered a memory leak that I suspected was the result of poor *var* scoping, but I couldn't locate the offending code. In the past, I've used [varscoper](https://github.com/mschierberl/varscoper) (sometimes in conjunction with [CodeChecker](https://github.com/wellercs/CodeChecker)) to locate this type of error. Both have worked, and are very good tools, but I never worked out a system for making them easy to integrate in my day-to-day development.  I had seen the [CFLint](https://github.com/cflint/CFLint) project, but the need to download jars and use Maven was a bit intimidating - until I actually gave it a shot.<!--more-->

Before I go over the setup process, for those who might be similarly intimidated by it, I won't bury the lead. CFLint is an amazing tool. You should use it. I was extremely impressed once we got it running - it provides a wealth of valuable information and is extremely configurable.

So, here was my process:

1. I started with the ["How Do I Install This Tool?"](https://github.com/cflint/CFLint/wiki/How-Do-I-Install-This-Tool%3F) Guide from CFLint, which first directed me to:
2. ["Maven in Five Minutes"](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html). - Because I wasn't looking to actually learn Maven, all I needed to go over here was the Installation section. So don't be intimidated - just get the download you need: [http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi) - In my case, I downloaded apache-maven-3.3.9-bin.zip.
3. From there, I followed the [installation guide](http://maven.apache.org/install.html). This may not be the best practice, but I have a `tools` folder where I download/manage my tools like the Android SDK and ant - that's where I unzipped Maven.
4. Following the instructions, I added Maven's bin to the PATH environment variable, via my `.bashrc`

    <pre class="highlight">
    export MAVEN_HOME="$HOME/www/tools/apache-maven-3.3.9"
    export PATH=$PATH:$MAVEN_HOME/bin</pre>	
5. I restarted iTerm, and `mvn -v` worked, as the instructions indicated. At this point, we're basically done with Maven, and it's back to Step 2 of the "How Do I Install" guide.
6. I decided to store CFLint in the same `tools` folder as Maven, so that's where I downloaded the repo. Following the instructions, I browsed to the root of the repository and ran `mvn clean install` - and lots of downloading and building occurred. Success.

At this point, CFLint is installed (you still can't use it from the command line, but we'll get to that). To get an idea of how it works, in terminal, navigate to CFLint's `/target` folder. From there, run `java -jar CFLint-0.6.1-all.jar -ui` (for whatever version of CFLint you have). This brings up the UI, from which you can select a folder directory to scan, then an output format (I selected html). With that, it runs - and it's fast.

In this case, it will create an file (`cflint-result.html`) within `/target` folder (probably not what you want to do every time, but fine for looking at the functionality). It will open the HTML result file in the browser and you can review the errors/warnings/info that it returns. I found it extremely helpful (and located some unscoped variables as well)[^1]. 

For CFLint to really fit with my workflow, this isn't quite enough. When I'm in a project, I want a fast, easy way to have the code parsed for errors. Running the jar and browsing to the folder is a nuisance. I need to be able to run it from the command line, and fortunately, setting this up is just one additional step.

The path to the actual CFLint executable is located within the CFLint folder, here: `CFLint/target/appassembler/bin/cflint`. All you need to do is [create a symlink](http://apple.stackexchange.com/questions/115646/how-can-i-create-a-symbolic-link-in-terminal) from wherever you store your binaries to this location. I keep most of my binaries in `/usr/local/bin`, so I navigated to that directory and ran:

```text
ln -s ~/www/tools/CFLint/target/appassembler/bin/cflint cflint
```

Now, I can run the `cflint` command from anywhere. So, while working on a project, I am able to run `cflint -folder .` and get a nearly immediate report.[^2]


<hr />
[^1]: CFLint returns a lot of information, probably way more than you need, but from what I understand, you can configure the rules you want run, etc. I might do a future post on the configuration options, depending on how deeply I dive into them - for now, I don't mind ignoring the information that I don't need.

[^2]: This might be helpful for some people. There is a [plugin for Sublime Text which utilizes CFLint](https://packagecontrol.io/packages/SublimeLinter-contrib-CFLint). Once you have CFLint up and running, it's not too much more work to get it set up as well. I think the only other thing it required is the [SublimeText/ColdFusion](https://github.com/SublimeText/ColdFusion) plugin. I tried it out for a while, but it didn't work for me. I know others like using it, so there you go.

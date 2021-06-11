---
date: 2019-05-31
published: true
title: "How to Install and Manage Multiple Versions of Java (hint: jabba and jEnv)"
layout: post
tags: [java, jabba, jEnv]
---
One of my goals for the coming months is to improve my proficiency with Java; that is, to put some real knowledge behind my current patchwork understanding, pieced together over the years via trial-and-error.
<!--more-->

Before diving into Java training courses on Udemy, Youtube, etc, I wanted to make sure that I had a way to use different versions of the JDK on my machine. This was due, at least in part, to the [changes Oracle made to their Java licensing](https://blog.jetbrains.com/idea/2018/09/using-java-11-in-production-important-things-to-know/) and release cycle. There are now a variety of Java versions and vendor distributions available, and I wanted to find a way to easily switch between different versions of the JDK without needing to actually update my system's version of Java or manually set `JAVA_HOME` each time.

Basically, the same way I can use different versions of Ruby, Node, and CFML, thanks to [rbenv](https://github.com/rbenv/rbenv), [nvm](http://nvm.sh), and [CommandBox](https://commandbox.ortusbooks.com/), respectively, it seemed I should be able to easily use different versions of Java, and not be tied to my system's default version. So, like a good developer I did some Googling and stumbled across [this helpful StackOverflow post](https://stackoverflow.com/questions/52524112/how-do-i-install-java-on-mac-osx-allowing-version-switching), and ultimately settled on using both **[jabba](https://github.com/shyiko/jabba)** and **[jEnv](http://www.jenv.be/)** - the former to install versions and the latter to configure them on a per-directory basis. Here's how it works:

## jabba to install different JDK versions

**jabba** is a cross-platform Java Version Manager. I found that it really excels at streamlining the process of installing different versions of Java. You can find the installation instructions along with some very helpful documenation on the [jabba Github repo](https://github.com/shyiko/jabba#installation).

Once installed, you can list available versions of the JDK via `jabba ls-remote`:

```bash
$ jabba ls-remote
1.12.0
1.12.0-1
1.6.65
adopt@1.12.33-0
...
zulu@1.7.95
```

Right now, there are over 100 results, literally from A-to-Z (that is, Adopt OpenJDK to Zulu OpenJDK). 

To install a particular version, such as Corretto, you run:

```bash
$ jabba install amazon-corretto@1.11.0-3.7.1
#installed to ~/.jabba/jdk/amazon-corretto@1.11.0-3.7.1/
```

On Linux/Mac, the JDK version is downloaded/installed to `~/.jabba`. Remember this, because you'll need to know where these are when using jEnv.

Versions of the JDK not readily available, such as Oracle's 11.0.3, which now requires an account to download, can be manually installed via a URL or file location. Here's how I installed Oracle's, after downloading it:

```shell
$ jabba install oracle@1.11.0-3=tgz+file:///Users/MYUSER/Downloads/jdk-11.0.3_osx-x64_bin.tar.gz
```

### Using jabba with Intellij on Mac

When I tried to add the jabba-installed JDKs to Intellij as SDKs, I ran into a small issue - because they're located in a hidden folder, they're not readily selectable via the file browser. Unsurprisingly, I wasn't the first person to have this issue, [a Reddit post in r/java](https://www.reddit.com/r/java/comments/a1rfvx/jabba_intellij_on_mac/) pointed me to two answers:

- Within Intellij, when browsing for the JDKs in `~/.jabba`, you can type **Command-Shift-Period** (⌘+⇧+.), which should reveal the hidden files/folders, making them possible to select. Apparently this doesn't work in all Intellij versions, so there's another option.
- In regular Finder, navigate to the folder `~/.jabba/jdk/JDK@VERSION/Contents/Home` - you should then be able to drag this into the Intellij Finder window, in order to use/select it. 

## jEnv for per-directory JDK assignment

With installing JDK versions sorted out, my next goal was to make managing and using them straightforward. A primary concern was the ability to assign a JDK version to a project/directory and then not have to think about it again. For me, **jEnv** nailed it.

So, what is [jEnv](http://www.jenv.be/)? To pull a quote from its website:

> jEnv is a command line tool to help you forget how to set the JAVA_HOME environment variable 

So, while it doesn't install different versions of Java, jEnv makes it much easier to manage the versions already on your machine (like those installed by jabba). Just a note that, unfortunately for Windows users, it is not currently cross-platform. 

After installing jEnv, I needed to add the JDKs from jabba (and from my machine). This is done using the `jenv add` command and providing path to the JDK version:

```bash
# Add a jabba JDK
$ jenv add /Users/MYUSER/.jabba/jdk/oracle@1.11.0-3/Contents/Home/
oracle64-11.0.3 added
11.0.3 added
11.0 added

# Add the system JDK
$ jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/
oracle64-1.8.0.181 added
1.8.0.181 added
1.8 added

# List JDK versions
$ jenv versions
```
Now - and this is what I was most excited about - I can set a Java version for a directory and then I don't need to think about it again. Here's how it works:

```shell
$ java -version
java version "1.8.0_181"

$ jenv local oracle64-11.0.3

$ java -version
java version "11.0.3" 2019-04-16 LTS
```

This works by adding a `.java-version` file to the directory, which contains the specified version. jEnv automatically ensures that this version is used within the folder (and subfolders). Set-it-and-forget-it.



## Wrapping it up

I was ultimately very happy with this solution - it enables me to do some initial Java configuration for a project and then just forget about it and get back to the code. That said, I'm no Java guru. If you've got a better approach or see problems with this one, please let me know.
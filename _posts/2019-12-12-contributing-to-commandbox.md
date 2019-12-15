---
published: true
title: "Contributing to CommandBox - Steps for Building and Developing Locally"
layout: post
tags: [commandbox,cfml,coldfusion]
---
One of the appeals of open source software is that anyone can contribute. When you encounter a problem with an open source project, beyond simply reporting the bug, you have the means of resolving it - anyone can send a PR. Having recently gone through this process with CommandBox, I thought it might be helpful to share the steps for contributing.
<!--more-->

What follows is a brief guide to running your own fork of CommandBox, so that you can develop locally, before submitting a pull request to merge your changes back into the main project. 

Credit where it's due, this primarily consists of me recounting the instructions and insights that [Brad Wood](https://twitter.com/bdw429s) provided while I put together a PR for a [bug that I had reported](https://ortussolutions.atlassian.net/browse/COMMANDBOX-1077). 

## Requirements

I think these are pretty safe assumptions, but just to be clear, for the steps outlined here, you will need to have:

1. [CommandBox installed](https://commandbox.ortusbooks.com/getting-started-guide) on your machine
2. A [Github](https://github.com) account

## Steps to Build and Develop CommandBox Locally


1. You'll want to start with the bleeding edge release of CommandBox - this ensures all jars, etc, are up to date and will make merging your changes smoother. To ensure you have the latest version, start CommandBox and run:

	```bash
upgrade --latest
	```

	You may need to download and install a new CommandBox binary. If that's the case, just follow the instructions provided: download the new binary, exit Commandbox, replace the old binary, and run `box` again. Done.

2. Go to the Github repository for CommandBox and fork it: [https://github.com/Ortus-Solutions/commandbox](https://github.com/Ortus-Solutions/commandbox)

3. Clone your fork of CommandBox locally. In my case, I navigated to the folder I do development in and ran the following command:

   ```bash
   # you'll want to update this for your fork
   git clone git@github.com:mjclemente/commandbox.git
   ```

4. In the following steps, you'll need to know where your CommandBox home folder is. If you don't know, here's how to find out - within the Box shell, run:

   ```
   open ${cfml.cli.home}
   ```

    This will take you to the folder. Mine was here: `~/.CommandBox`

5. Make sure CommandBox is stopped now. It shouldn't be running during the next few steps.

6. If you've installed CommandBox system modules ([cfconfig](https://www.forgebox.io/view/commandbox-cfconfig), [commandbox-bullet-train](https://www.forgebox.io/view/commandbox-bullet-train), [dotenv](https://www.forgebox.io/view/commandbox-dotenv), [etc](https://www.forgebox.io/type/commandbox-modules)), you may want to back them up now. To do this, make a copy of their `box.json` file, which is located under the CommandBox home directory, in the `cfml` folder. For reference, mine was here:  `~/.CommandBox/cfml/box.json`. Save that copy somewhere safe.[^1]

7. Ok, now we get to the actual changes needed to develop locally. The next step is to delete the `cfml` folder found under the CommandBox home. Because mine was here, `~/.CommandBox/cfml`, I ran the following command:

	```bash
rm -rf ~/.CommandBox/cfml
	```
	You may wish to handle this less destructively - simply renaming the folder or manually moving it to the trash, so you can recover it later.

8. Next, replace the directory you just deleted with a symlink to the `src/cfml` folder located within your fork of the project. On Mac I ran the following:

	```bash
ln -s ~/path/to/my/fork/commandbox/src/cfml ~/.CommandBox/cfml
	```

	So, now the `cfml` folder in the CommandBox home points to the `src/cfml` folder in your Git repository. That's where you'll be developing.

9. If this is your first time working on CommandBox, there shouldn't be any uncommited changes in your repository, but you should check by running: `git status`. Uncommited changes may be lost.

10. You can now start CommandBox again. It may take a moment to initialize, download libraries, etc.

11. It's possible that the previous step will result in changes to your local Git repository, because it's symlinked. To make sure that everything is clean for development, again run `git status` and clean out any unstaged changes that resulted from the binary starting up.

12. Ok, that's it. Updates made within your Git repository will be reflected in the local version of CommandBox that you're running. Note that you'll need to run `reload` for CommandBox to pick up your code changes while it's running.

13. The last step is, hopefully, to contribute to the project. Fix a bug or add a feature, push the changes to your Github repository, and then [send a PR](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork). Every little improvement makes CommandBox better, for yourself and for the CFML community!

One last note that Brad mentioned - if you ever switch to a new CommandBox binary, you may lose any uncommitted changes sitting in your Git repo. Just something to be aware of.

## Rolling back to stable

Ok, so you've messed around with running your fork of the bleeding edge of CommandBox, but now you want to switch back to the stable release. How do you do it? It's very straightforward:

1. Make sure CommandBox isn't running.
2. Delete the `cfml`, `engine`, and `lib` folders in the CommandBox home directory.
3. Replace the CommandBox binary with the binary for the stable version. You can find them all [here](https://downloads.ortussolutions.com/#/ortussolutions/commandbox/).
4. Start up CommandBox and it will re-extract and reset everything. You're back to stable.



____

[^1]:One approach would be to save/track this type of configuration file in a separate Github repo. This is a common approach with [dotfiles](https://github.com/mathiasbynens/dotfiles). To reinstall these system modules, restore the `box.json` file to its proper place and then within CommandBox run `package install --system`.
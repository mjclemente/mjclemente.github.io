---
published: true
title: Getting Started with Jekyll (Part 1, Wrestling with Ruby)
layout: post
tags: [blogging,jekyll,ruby]
---
This was supposed to be a post complaining about the standards, or lack thereof, applied by ThemeForest, to the Wordpress Themes they well. As I went to write it I noticed that TinyPress.co's SSL certificate had expired, so it seemed as good a time as any to dive into Jekyll and handle this static blogging with Github Pages myself.

What follows isn't a guide; it's just my log of the 600+ mistakes it took for me to get started using Jekyll. As such, it contains a lot of dead ends before I got it all working. In hindsight, it seems that I made things significantly more difficult by starting with TinyPress, instead of just RTFM for Jekyll and Github Pages myself. So much for shortcuts.

I started out well enough, using the [Jekyll Installation Docs](http://jekyllrb.com/docs/installation/). They said the best way to install it was using RubyGems, which I hadn't used before, but seemed easy enough, so I ran the command:

	gem install jekyll

And promptly got the response:

	ERROR:  While executing gem ... (Gem::FilePermissionError)
    	You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory

With a little Googling and reading on the [Jekyll Troubleshooting](http://jekyllrb.com/docs/troubleshooting/) page, it turns out this is because, unsurprisingly, the system defaults to using the version of Ruby installed by Apple. The first workaround I tried was changing the location where the gem would be installed. I try to use `/usr/local/bin` for the binaries I install anyway, so this made sense:

	gem install -n /usr/local/bin jekyll

Same error. And while the troubleshooting pages recommend running commands such as `sudo gem update --system`, `sudo gem install jekyll`, and `sudo gem install -n /usr/local/bin jekyll` I didn't want to mess around with the system version of Ruby, and knew that I shouldn't need to use `sudo` for this, if I set it up right.

The next step was to, following the advice on this [SO page](http://stackoverflow.com/questions/14607193/installing-gem-or-updating-rubygems-fails-with-permissions-error), install a Ruby version manager. I went with [*rbenv*](https://github.com/rbenv/rbenv).

I've had a few bad experiences with Homebrew, so I decided to install it manually:

	git clone https://github.com/rbenv/rbenv.git ~/.rbenv
	cd ~/.rbenv && src/configure && make -C src

I followed the [installation instructions provided](https://github.com/rbenv/rbenv#installation) through step 4 and it went smoothly (I added it to my `$PATH`, ran the `init`, and followed the instructions.) I ran: `type rbenv` to confirm the installation and it comes back as a function, so I thought I was golden. I try running the command to install Jekyll again... and get the same error.

Genius that I am, I had installed the Ruby version manager, but not any new versions of Ruby, so of course it was still using the system version. I go back to the instructions, read step 5, which was marked as *(Optional)*, and realize that I need to [install ruby-build](https://github.com/rbenv/ruby-build#readme), in order to easily add new versions of Ruby (as alternatives to using the system version). So, I run the installation:

	git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

Run the command to list available versions of Ruby to install:

	rbenv install -l

And then install the most recent stable version:

	rbenv install 2.2.4

Things go smoothly. I see that they're being installed to `~/.rbenv/versions/`. I run the command to set the newer version to be used globally:

	rbenv global 2.2.4

At this point, I needed to restart the terminal, and then finally, I was able to run the Jekyll install command successfully:

	gem install -n /usr/local/bin jekyll

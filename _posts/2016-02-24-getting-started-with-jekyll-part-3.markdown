---
published: true
title: Blogging with Jekyll (Part 3, Actually Using Github Pages)
layout: post
tags: [blogging,jekyll,ruby]
---

This is the final installation in my three part attempt to get up and running with Jekyll and Github Pages. [Part 1](/2016/02/23/getting-started-with-jekyll-part-1.html) recounted my wrangling with Ruby, [Part 2](/2016/02/24/getting-started-with-jekyll-part-2.html) outlined the missteps I took while getting Jekyll to run, and this will be my overview of Github Pages.

This time, I actually read the docs. From Github Help: [Setting up your Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-pages-site-locally-with-jekyll/).

They recommend using the Github Pages gem, and the Bundler gem. Obviously, I didn't have either of these installed. So I started with Bundler:

	gem install -n /usr/local/bin bundler

There are then a set of instructions for installing the Github Pages Gem, which are pretty straightforward. Create `Gemfile` in the root of the repo. Add these lines:

	source 'https://rubygems.org'
	gem 'github-pages'

Finally, run `bundle exec jekyll build --safe`, which I did. Of course, it didn't run successfully. Here was the error:

	Could not find gem 'github-pages' in any of the gem sources listed in your Gemfile or available on this machine.
	Run `bundle install` to install missing gems.
	
At least the message was instructive, so I ran it: `bundle install`, and the gems were downloaded and installed within *.rbenv* folder. If you run `gem environment` you can get some more helpful information about your gem setup.

At this point, I ran `jekyll serve`, and again, got an error:

	Dependency Error: Yikes! It looks like you don't have kramdown or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- kramdown' If you run into trouble, you can find helpful resources at http://jekyllrb.com/help/!

After doing a little more reading of the documentation, I realized that the command I needed to use, in order to ensure that all the appropriate gems were available, was: `bundle exec jekyll serve`, which ran without error.

However, when I accessed the site locally, the posts still were not being listed on the homepage. So, I took the error and warning messages that I was receiving when I ran the command, one at a time. 

The first warning, when running the serve command, was `'Layout 'nil' requested in atom.xml does not exist.'`, so I dug a little further into where this was coming from. This was probably a consequence of setting the site up through Tinypress.co. I updated the line in atom.xml from `layout: nil` to `layout: null`, which resolved the warning, but not the post list issue.

The second warning that resulted from the serve command was mentioned in the previous post:
	
	Message: "Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration file."

I pasted the message into Google, which is always a helpful approach, and that led me to this [helpful answer](https://teamtreehouse.com/community/jekyllpaginate-gem). I needed to update my *_config.yml* to include `gems: [jekyll-paginate]`, so the final two lines looked like this:

	gems: [jekyll-paginate]
	paginate: 5

From there, I restarted the terminal, and ran `bundle exec jekyll serve`, which worked. The local server was functioning correctly and my posts were being displayed.

It was a long process, but definitely worth it. I can now write and develop locally, and push my changes and posts to Github when I'm ready. I definitely took the long way around, in getting it set up, but I am very impressed and happy with the Jekyll/Github Pages blogging setup.

 
 
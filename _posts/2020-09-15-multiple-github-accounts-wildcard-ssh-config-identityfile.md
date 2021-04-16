---
published: true
title: "A Note on Misconfiguring my SSH Config When Setting Up Two Github Accounts"
layout: post
tags: [github, git]
typora-root-url: ../../blog.mattclemente.com
---

Recently, I set up a demo Github account. I already had a demo GitLab account. So now I have two GitHub accounts, two GitLab accounts, and a Bitbucket account for good measure.  It took me a few tries to get the multiple accounts working correctly, so these are my notes, in the event that I forget, or need to set it up again.

<!--more-->

**tldr;** You should explicitly list all Git Hosts in your `~/.ssh/config`. If you include a `Host *` listing with an `IdentityFile` entry, it may cause issues.

* TOC
{:toc}

## Two Accounts... Why?

There are a handful of reasons to have multiple GitHub/GitLab/Bitbucket accounts. The most common is probably having separate work and personal accounts. In my case, I wanted to set up a separate account to use for live coding. Of late, I've been [live streaming each week](https://www.youtube.com/channel/UC09HBVzOOyx1bdgRgo2CB4A) while exploring new (to me) tools and services. A separate, demo GitHub account provided better code organization, and from a security perspective, minimized the risk if I accidentally revealed an API key or some other secret while broadcasting.

Regardless of your reason or the remote repository provider (GitHub, GitLab, Bitbucket) that you're using, the general approach to setting up multiple users is the same.

## Resources

So, this topic has been covered in dozens of other blogs and Q&A posts. Here are a few I referenced while sorting this out:

* [A Practical Guide to Managing Multiple GitHub Accounts](https://medium.com/the-andela-way/a-practical-guide-to-managing-multiple-github-accounts-8e7970c8fd46):
  This is the primary post I used, and I found it thorough, clear, and helpful. If it weren't for the issue with my `~/.ssh/config`, discussed below, following the steps here would have been sufficient.
* [Stack Overflow: Multiple github accounts on the same computer?](https://stackoverflow.com/questions/3860112/multiple-github-accounts-on-the-same-computer)
  Filled with helpful information and some thorough responses.
* [Quick Tip: How to Work with GitHub and Multiple Accounts](https://code.tutsplus.com/tutorials/quick-tip-how-to-work-with-github-and-multiple-accounts--net-22574)
  Old Jeffrey Way post from Tuts+, that includes a screencast.

## The Setup (in brief)

I'm not going to retype the entire process; it's already covered in the resources listed above, if you're interested in the specifics. However, so that I remember when I come back to reference this in six months, here's a quick overview of the setup process:

1. **Create Separate SSH Keys**: This is the actual key to the process. You use `ssh-keygen` to create a key for each account you want to set up, giving the generated keys different, clear names; for example `demo_github_com_rsa`



2. **Add to GitHub**: You'll need to copy the public key that was generated (i.e. `demo_github_com_rsa.pub`) and follow the [instructions for GitHub](https://docs.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account) (or [GitLab](https://docs.gitlab.com/ee/ssh/README.html#adding-an-ssh-key-to-your-gitlab-account), or [Bitbucket](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/#SetupanSSHkey-#installpublickeyStep3.AddthepublickeytoyourAccountsettings)) to add it to your account. You'll need to do this for all the accounts that you are creating.



3. **Configure Hosts**: For each account you're setting up, you add a Host entry in your `~/.ssh/config`. The key here is that you use different Host's for the different accounts. So, I use `github.com` for my primary account, and `github.com-demo` as the Host for the demo account. I still haven't settled on if that's the naming structure I want to stick with - but it's a starting point.



4. **Use Correct Host When Connecting**: When I'm connecting to the demo account, I need to use the modified Host. So, for example, when adding a remote, it would look like this:

   ```shell
   git remote add origin git@github.com-demo:USERNAME/demo.git
   ```

   Notice the use of `github.com-demo` where it would normally be just `github.com`. You can verify that your different account Hosts work by running `ssh -T github.com-demo` - just replace the host with whatever your used for your own.

Ok, so, this all may or may not seem straightforward to you. I thought I had followed the instructions correctly, but I kept not being able to connect to my demo account. Instead, it kept trying to use my primary account. Eventually, I tracked down the culprit...

## The `~/.ssh/config` Misconfiguration

The above posts would have been all I needed, were it not for one configuration issue. At the end of my config file, I had the following wildcard listing:

```text
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
```

Here's the short diagnosis - the wildcard `Host *` causes problems; specifically, its `IdentityFile` entry. To resolve it, I had to 1) remove the `IdentityFile` from the wildcard Host, and 2) add an entry in the config file for each Host I connect to.

Here's the slightly longer explanation, that I learned after a lot of reading. Most Host options can only be specified once, with [the first occurrence being used](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client#interpretation-algorithm). This is why wildcard configurations should be put last - to provide default values. However, *it's possible to have multiple identity files*; from the [Linux man pages](https://man7.org/linux/man-pages/man5/ssh_config.5.html):

>  Multiple IdentityFile directives will add to the list of identities tried (this behaviour differs from that of other configuration directives).

You can see this, if you run the command `ssh -v -T github.com-yourHost`. The option `-v` provides verbose debugging output. When using a wildcard Host with an `IdentityFile`, in the output you can see that both of the identity files are included in the exchange. For me, this lead to my default Identity File being used, instead of the one from the custom Host.

So, to wrap it up, the final setup could look something like this, still using a wildcard Host, just not for the Identity Files:

```text
# --- For Demo GitHub Repo ---
Host github.com-demo
  IdentityFile ~/.ssh/demo_github_com_rsa
  IdentitiesOnly yes

# --- For Demo GitHub Repo ---
Host github.com
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes

# Defaults
Host *
  User git
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
```

___
**Update - 04/16/2021**: I've updated this to include the `IdentitiesOnly yes` option. As explained in [this Stack Overflow answer](https://stackoverflow.com/questions/7927750/specify-an-ssh-key-for-git-push-for-a-given-domain/7927828#7927828), this option prevents the use of default ids. In this specific example the option is not needed. However, in more advanced setups, if `ssh` intelligently matches hosts, this option ensures only the specified `IdentityFile` is used, and that another is not appended.
___

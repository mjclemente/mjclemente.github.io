---
published: true
title: Versioning With Git Tags
layout: post
tags: [git,github,tags]
---
Admission: I've only recently started using tags to version my Git repositories. I had the (mistaken) idea that the process was difficult and never bothered learning the specifics. Working more with open source projects eventually lead to the realization that tags and versioning provided a developer-friendly road-map for interacting with the codebase. When I decided to familiarize myself with Git's tagging workflow, I was pleasantly surprised how simple it was.
<!--more-->

The following is an outline of the process I now use to version my Github repositories with Git tags:

1\. Development is done on the `develop` branch of the repository. (Not using branches? You should be, but for now you can just ignore the first two steps.)

2\. When you're ready to release a version, checkout `master` and merge `develop`.

```shell_session
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ git merge develop
Updating xxxxxxx..xxxxxxx
Fast-forward
```
 
3\. Tag `master` with the version you want to assign. You can see the syntax in the example below. The `-a` means that this is an annotated tag, followed by the version. Annotated tags include a message, which you specify with `-m`.

```shell_session
$ git tag -a v0.0.1 -m "Put your message about the version here."
```

4\. Push your commits. Note, this does not push the tag, just the commits.

```shell_session
$ git push
```

5\. Push the tag. *Git doesn't automatically push tags, when you push commits*. You'll forget this, because it's not intuitive, but there it is. 

```shell_session
$ git push origin v0.0.1
```

6\. That's it. Your Github repository should now show both the tags and releases.
![Git tags and release versions](/public/assets/images/git-tags-and-release-versions.png)

For reference, I found [this Stack Exchange post](https://softwareengineering.stackexchange.com/questions/165725/git-branching-and-tagging-best-practices) helpful, as well as the [Git Basics - Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging) post.

<hr />

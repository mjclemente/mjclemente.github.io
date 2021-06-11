---
date: 2016-12-04
published: true
title: Switching from LastPass to 1Password and Keeping Your Folders
layout: post
tags: [lastpass,1password]
---
As evidenced by the dearth of posts, life, both personally and professionally, has been extremely busy of late. I don't see it slowing down any time in the short term. However, I wanted to document my process of moving from LastPass to 1Password, and converting my LastPass folders into 1Password tags. This is more for my own reference than anything else. The process wasn't terribly smooth, but I've been thrilled with making the switch.
<!--more-->

Everyone has reasons for changing password managers; mine aren't particularly relevant to anyone but myself. So they've been relegated to a footnote.[^1] Now let's dive in.

[1Password](https://1password.com/) is always near the top of the discussion when it comes to password managers, but its maker, AgileBits, is primarily focused on Mac/iOS.[^2] It is the password manager that security guru Troy Hunt uses, and he actually provided [a basic guide for switching from LastPass](https://www.troyhunt.com/logmein-now-owns-lastpass-heres-how-to/) following the the company's acquisition by LogMeIn. As a basic guide for migrating, it's very helpful, if a little dated, and if you don't really use the LastPass folders, it should be all that you need. 

A few notes/disclaimers:

1. I am using the 1Password subscription service, not the standalone install. I believe the standalone install supports folders; the subscription service only uses tags. I believe the rest of the migration process is the same.
2. I didn't use the "Identities" feature within LastPass, so I'm not sure how that factors into the migration.
2. I'm doing this on a Mac. I haven't used 1Password for Windows.
4. A warning, copied and pasted from Troy's guide:

> Firstly, this process is going to involve handling your credentials in plain text. Don’t do this at in internet cafe, don’t do it in front of your friends and don’t do it on any machine you don’t fully trust so that means running up to date anti virus (and keeping in mind that’s frequently ineffective these days) and not putting the file we’re going to generate anywhere that then gets backup up to other locations or is handled by other processes.

Without further ado, let's move our data from LastPass to 1Password:

1. Export your data. Here are the [instructions from LastPass for exporting your data](https://lastpass.com/support.php?cmd=showfaq&id=1206). It's basically LastPass Icon > More Options > Advanced > Export. If for some reason your data is displayed in the browser instead of downloaded as a .csv, you can copy and paste it all into a text editor and save it as a .csv. We'll call it *lastpass.csv* and put it on our Desktop.
2. *If you don't care about keeping your folders*: you can just import this directly. In fact, this is what 1Password guides you to do, if you go to 1Password > File > Import > From LastPass.
	![1Password import from LastPass](/public/assets/images/1password-import-from-lastpass.png)
3. *In order to keep your folders*: The magic starts with this [blog post on the AgileBits forum](https://discussions.agilebits.com/discussion/30286/mrcs-convert-to-1password-utility/p1) discussing MrC's Convert to 1Password Utility[^3]. 
4. Download the zip for the [Stable Bits](https://www.dropbox.com/sh/dciv3ne1xc1akg9/AAA-_VvYNLiWmUyaSHHz3CZ8a?dl=0) version of the tool to your Desktop.
5. When I ran it, the version was 1.08 (*convert_to_1p4_1.08*). Obviously, this process could change from version to version.
6. Unzip the package. You'll get a folder called *onepassword-utilities* that contains another folder *convert_to_1p4*.
7. From terminal, cd into the directory:

	<pre class="highlight">
	$ cd ~/Desktop/onepassword-utilities/convert_to_1p4</pre>

8. Now unfortunately you can't use the included *AppleScript_Conversion_Helper*, as recommended in the guide, because we want to set a few options regarding the folders that need to be done manually.
9. As the documentation explains, the format of the command we'll need to run is 

	<pre class="highlight">
	$ perl convert_to_1p4.pl &lt;converter&gt; &lt;options&gt; &lt;export_text_file&gt;</pre>
 
	a. In this case, the converter we're using is `lastpass`, which is self-explanatory.
	
	b. Our options are: `-a -f`. The former makes sure that we get non-stock fields imported as custom fields, and the latter is what converts our folders to tags (for the standalone install of 1Password, it may actually create the folders, I'm not sure).
	
	c. There's also an option to  add a tag to all items imported: `-t "name-of-tag"`. I used this just organize the import, then removed the tag once I was comfortable with the result. Obviously, it's optional.
	
	d. And the export text file is `~/Desktop/lastpass.csv`, or wherever you downloaded the file to.
	
10. This results in the following command:

	<pre class="highlight">
	$ perl convert_to_1p4.pl lastpass -a -f -t "lastpass-import-the-date" ~/Desktop/lastpass.csv</pre>

11. The result is that a file named *1P_import.1pif* will be generated on your Desktop. This contains your data to be imported into 1Password.
12. To import the data, use 1Password > File > Import > From 1Password and select the option "Using the Mac app". 
	![1Password import from 1Password](/public/assets/images/1password-import-from-1password.png)

13. When you select the *1P_import.1pif* file that was generated, your data will be imported and you'll have the option to view it.
14. Along with seeing all your data, you'll notice on the left, that your folders have been converted to tags.
	![LastPass folders as 1Password tags](/public/assets/images/1password-folders-as-tags.png)

At this point, your accounts, along with their folders have been transferred.[^4] 

*The following is only applicable to the 1Password subscription service, as the standalone install supports folders*. The tags created based on your folders are simply the text of the full folder path. This won't break anything in 1Password, but I still prefer to complete the migration from folders to tags. That is, rather than having a tag named `Personal/Development/Wordpress`, I want to have three tags - one for `Personal`, one for `Development`, and one for `Wordpress`. It took me a little while to figure out how to do this, but it really isn't all that hard.

1. Choose one of the tags that you want to add. (In my case, I'll add `Development` first.)
2. Select one of the logins that is currently using the "folder path" type tag. (I'll start with one that's using `Personal/Development`.)
3. Click the *Edit* button on the lower right corner of the item details.
4. Click into the *tags* field, and enter the new tag you are adding.
	![1Password Add New Tag](/public/assets/images/1password-add-new-tag.png)
5. When you click *Save* in the lower right corner, the new tag will appear in the list of tags on the left.
6. Now, you can click on the original tag (`Personal/Development`) in the left, which will display associated logins in the right.
7. Click on one of the logins in right column, highlighting it. Then type the shortcut `⌘ A` to select all (or use the menu Edit > Select All).
8. You can now drag all of the highlighted options to the new `Development` tag that you created.
9. Repeat the above process for the next tag (in my case, `Personal`). 
10. Once I have added the logins from the original `Personal/Development` to the new `Personal` and `Development` tags, I can remove the original tag (which is as simple as right-clicking and selecting the option to Delete).

A final note, about searching based on tags in 1Password - if you only want to search/filter based on one tag, it's obviously as easy as selecting it from the tag list on the left. If you want to search/filter by multiple tags (for example, viewing logins that are tagged as both `Personal` and `Development`) it's possible, and actually quite robust, but not intuitively located. To view the search options

1. Click into the search field on the top.
2. Then click on the small, down arrow **&or;** on the left.
3. This brings up a menu, where you can select "Show Search Options".

	![1Password Search Options](/public/assets/images/1password-search-options.png)
4. Once the search options are showing, you can select the field type dropdown, to see the range of searches possible. Searching by tag(s) is one of these options.
![1Password Search Options List](/public/assets/images/1password-search-options-list.png)

That's it; we've migrated from LastPass to 1Password, converted our folders to tags, and found out how to do advanced searches by multiple tags. Once you're done and satisfied, don't forget to permanently delete (`Alt+Cmd+Backspace`) the *lastpass.csv* and *1P_import.1pif* files, and [delete your LastPass account.](https://lastpass.com/support.php?cmd=showfaq&id=163).

I've been using 1Password for a few months now and I've been thrilled with the switch.
<hr />
[^1]: I signed up for LastPass in January of 2013, as part of a New Year's resolution to organize my online accounts. I researched other options, but ended up settling on LastPass because it was well reviewed, affordable, and provided cross-platform support. I was, generally, very happy with LastPass. I didn't leave because of the [LogMeIn acquisition](https://www.reddit.com/r/YouShouldKnow/comments/3o4p7b/ysk_lastpass_has_been_acquired_by_logmein_a/), the [data breach](https://blog.lastpass.com/2015/06/lastpass-security-notice.html/), or the [various](https://twitter.com/taviso/status/758074702589853696) [reported](https://blog.lastpass.com/2016/07/lastpass-security-updates.html/) [vulnerabilities](https://labs.detectify.com/2016/07/27/how-i-made-lastpass-give-me-all-your-passwords/).  My primary motivation was much simpler: it just got too buggy to use. (And yes, I tried uninstalling and reinstalling the binary/plugins/extensions. I made sure that I had the most recent version. I know this isn't the experience that everyone has with them, but I really didn't feel like spending a ton of time trying to debug it.)
[^2]: Though they do have a [Windows version](https://agilebits.com/downloads) in beta now.
[^3]: As he states in the post: *The conversion utility is not an AgileBits product, nor do I work for AgileBits. I'm a user like you who just loves 1Password.*
[^4]: Keep in mind that, as stated in the *Convert to 1Password Utility* docs, your attachments *will not* have been migrated. *So you will need to migrate your attachments manually*.
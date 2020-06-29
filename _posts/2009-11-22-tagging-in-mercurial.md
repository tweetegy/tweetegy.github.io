---
title: Tagging in Mercurial
author: jensendarren
layout: post
permalink: /2009/11/tagging-in-mercurial/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
  - Programming
tags:
  - hg
  - mercurial
  - release
  - tag
  - tagging
  - versioning
---
**Introduction**

If you are building software and want to release that software at regular intervals then why not tag each release in Mercurial? That way you can very easily rollback to that version of the software at any time. Whats more is that you can use your own tags rather than trying to remember the mercurial revision number that was current at the time of release.

There are many approaches to software versioning which will not be covered here. Check out this wikipedia article on <a href="http://en.wikipedia.org/wiki/Software_versioning" target="_blank">software versioning</a> for more details on the different approaches. The purpose of this article is to apply the versioning as a tag in your mercurial repository.

**Apply a tag in mercurial**

When using mercurial, you must always tag after a commit. So if you know that you want to tag your current working directory, you must commit it first and then tag it. Given this it obviously makes it very easy to tag a commit that you might have made a long time ago in the past. Bear this in mind when using the tag command in mercurial and you will understand clearly how this feature can be used.

This is how you would tag a revision in a mercurial repository, first decide what revision you want to tag and then run the following command. In this example it will tag revision number 45 with &#8220;2.0.5&#8243;

{% highlight bash %}hg tag -r 45 2.0.5
{% endhighlight %}

This command automatically commits this tag, so if you run &#8220;hg tip&#8221; in your terminal, you will see something like the following:

{% highlight bash %}changeset:   51:209937087918
tag:         tip
user:        Darren Jensen
date:        Sun Nov 22 13:10:47 2009
summary:     Added tag 2.0.5 for changeset b50b86512aac
{% endhighlight %}

Note that the changeset for &#8220;tip&#8221; is 51:209937087918 and the comment is &#8220;Added tag 2.0.5 for changeset b50b86512aac&#8221; (this comment is also added automatically by mercurial by the way). Given that I asked mercurial to apply the tag to revision 45, you can easily see that it is a commit made a while ago. The changeset indicated in the comment (&#8220;b50b86512aac&#8221;) does indeed refer to revision 45.

**Use a tag in mercurial**

So how to use this tag going forward? Here are a couple of ideas.

Use it to see the commit info at that time using the log command as follows:

{% highlight bash %}hg log -r 2.0.5
{% endhighlight %}

This will return the log entry that is associated with that tag, so in our example it would return the entry for changeset 45:b50b86512aac, as follows:

{% highlight bash %}changeset:   45:b50b86512aac
tag:         2.0.5
user:        A Developer
date:        Tue Jul 21 14:27:50 2009
summary:     Made some serious changes to the application and deployed it to production
{% endhighlight %}

Use it to clone the repository to the tag point (note that the tag itself would not be included since that is added after this point). So if you do a &#8220;hg tip&#8221; on this cloned repository, then your would see the log entry for changeset 45 but the tag and the changeset that commit the tag will not be there (changeset 45 will have the tag &#8220;tip&#8221; in this case!).

{% highlight bash %}hg clone -r 2.0.5 somerepo
{% endhighlight %}

Check out the Mercurial wiki for more details regarding <a href="http://mercurial.selenic.com/wiki/Tag" target="_blank">tagging in mercurial</a>, including how to deal with tagging conflicts, local tags (as opposed to &#8220;regular&#8221; tags which is covered in this article), removing tags and more.

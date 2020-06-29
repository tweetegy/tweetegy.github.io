---
title: 'Installing Rails gem fails with &#8220;undefined method `spec&#8217; for nil:NilClass&#8221; error'
author: jensendarren
layout: post
permalink: /2011/04/installing-rails-gem-fails-with-undefined-method-spec-for-nilnilclass-error/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
  - Rails Appplication Programming
tags:
  - bash
  - gem
  - install
  - rails
---
Today, while installing Rails 3, my computer suffered a power outage. This does not happen very often, in fact, this was actually due to the computer overheating! After pouring cold water over the keyboard, I restarted the computer and continuing the Rails 3 installation the process, but to my shock and horror, it failed with this message:

{% highlight bash %}ERROR:  While executing gem ... (NoMethodError)
undefined method `spec' for nil:NilClass
{% endhighlight %}

I&#8217;ll be honest with you, I was kind of expecting problems to happen since the computer did abruptly shut off during an installation process &#8211; but now what do I do!? Well, after a little Googling, the problem was solved. I found out that it&#8217;s due to the gem install cache folder which (due to the power interruption) now contained corrupted / empty gem files. My solution was to clear this cache and run the install again, as follows:

1.  Find out the location of your cache using: `gem env`
2.  List all the gems in that cache, using, for example: `ls -l /usr/lib/ruby/gems/1.8/cache/`
3.  Compare with successfully installed gems using `gem list`
4.  Remove the gems from the cache list that are missing from the gem list using the `rm` command
5.  Continue the installation, for example. `sudo gem install -v=3.0.6 rails --no-rdoc --no-ri`

So now if you have a power outage or any other external failure during a gem installation process you don&#8217;t need to panic. Just clear the cache and continue where you left off before you were rudely interrupted!

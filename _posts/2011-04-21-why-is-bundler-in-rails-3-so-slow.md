---
title: Why is Bundler in Rails 3 so slow?
author: jensendarren
layout: post
permalink: /2011/04/why-is-bundler-in-rails-3-so-slow/
image:
  -
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
  - Ruby Server Programming
tags:
  - bundler
  - gems
  - installation
  - rails
  - rails3
  - rubygems
  - ruiby
---
Actually, the title of this post is mis-leading a little so blame Google for bringing you here if this is not what you wanted!

What I will do is tell you this &#8211; Bundler is really slow on my machine! It takes more than **40 minutes** to install just a **single** gem! I have heard other complaints around the web-sphere but nowhere near as long as I have to wait (usually I find most people who have slow Bundler experience wait more than 3 minutes on average). OK, so I am not going to tell you why this is happening, but I *will* tell you a work around.

Instead of running:

{% highlight bash %}bundle install
{% endhighlight %}

&#8230;run it with the &#8211;local switch instead as follows:

{% highlight bash %}bundle install --local
{% endhighlight %}

If you want to install a new Gem, lets say, mongo_mapper, then add to your application Gemfile:

{% highlight ruby linenos %}gem 'mongo_mapper'
{% endhighlight %}

&#8230;and here&#8217;s the important part, install it the old fashioned way, using `gem install` and *then* run `bundle install --local`

{% highlight ruby linenos %}gem install mongo_mapper --no-rdoc --no-ri
bundle install --local
{% endhighlight %}

Note that I pass the &#8211;no-rdoc and &#8211;no-ri switch to gem install to make that a little (sometimes a lot) faster too. Now you can get back to building that app! Good luck!

---
title: Mixing Ruby in Bash
author: jensendarren
layout: post
permalink: /2012/09/mixing-ruby-in-bash/
seo_follow:
  - 'false'
image:
  -
categories:
  - Ruby Server Programming
tags:
  - -e option
  - bash
  - inline code
  - ruby
---
If your like me, I usually dread the idea of writing a bash script. Generally these days, I try to write any tasks using Ruby. Sometimes, however, it just seems that all I do is write lots of `Kernel.exec` or calls to `sh` method in Ruby! Othertimes, I just instintively start writing something in bash which is what I did today.

## The Original Problem

I needed a startup script that would quickly get a bunch of services up and running on my development machine so that I can get coding quickly and easily. It all worked fine, but when I executed the script multiple times new instances of Ruby would be started, which is not what I wanted. I generally wanted to *restart* the services instead. So I used the following to kill all my ruby processes (remember, I only use this script in my *development* environment!):

{% highlight bash %}rubypids=`pidof ruby`
sudo kill $rubypids
{% endhighlight %}

This bash script sets `$rubypids` to a list of, well, Ruby pids which looks like so `2566 2899 2908` and it turns out that passing that to `kill` will kill them all.

## The New Problem

The problem was that one of these pids was my instance of RubyMine that I *diddnt* want to kill! At that point I am thinking, damn *so how to do find and replace within a string in bash?*. After some unsucessful playing around with the `sed` command, I figured *why not just use Ruby to do it instead*. To my joy, Ruby works very well in bash! I used the `-e` option of the `ruby` command to do it. Here is the updated script:

{% highlight bash %}#Get all ruby pids and the rubymine pid
allrubypids=`pidof ruby`
rubyminepid=`pgrep rubymine`

#Use ruby to reject the rubymine pid
rubypids=`ruby -e "puts %w($allrubypids).reject {|pid| pid == $rubyminepid.to_s}"`

#Kill the rest
rvmsudo kill $rubypids
{% endhighlight %}

So I fell for an Array#reject solution simply because I know how to work with Arrays in Ruby very well and because it was the first option that came to mind. I love how simple and seamless it is to &#8216;pass&#8217; bash variables to Ruby and have the result of the Ruby execution passed back to bash. Maybe this can be useful again in the future when stuck with how to do something &#8216;simple&#8217; (using bash) &#8211; do it in Ruby instead!

---
title: 'Dissecting thesmallestrailsapp.com (Smallest Rails App: Newsflash)'
author: jensendarren
layout: post
permalink: /2014/03/dissecting-thesmallestrailsapp-com-smallest-rails-app-newsflash/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - bundler
  - rails
  - require
  - ruby
  - ruby load path
---
Since starting this series, the inevitable has happened &#8211; The Smallest Rails app has been refactored! In this post I want to cover the recfactorings to the first line which affects the first blog post in this series [][1].

To recap the first line was as follows:

{% highlight ruby linenos %}%w(action_controller/railtie coderay markaby).map &method(:require)
{% endhighlight %}

Now this has been changed to the following (2) lines (Note: since writing this post the first line has been removed (see conclusion below):

{% highlight ruby linenos %}require "bundler/setup"
Bundler.require
{% endhighlight %}

For clarity, here is the [commit diff][2] in Github for this change.

## Compare with Initial Commit

My first thought was that maybe this application was not using Bundler (before), but I can confirm that the first commit of this project was and even had the line ` require 'bundler/setup'` so its been used all the time. For your information the initial commit of this project has a commit SHA of &#8220;2215c1a&#8221; and if you want to see how this looked just simply clone the project and checkout that commit to detach your code from HEAD, as follows:

{% highlight bash %}git clone git@github.com:artemave/thesmallestrailsapp.com.git
git checkout 2215c1a63700f21d54d0d16466f5527932ac8df6
{% endhighlight %}

As you can see the application looks quite different from what it does now, but it does appear to be using Bundler. Perhaps another interesting series of posts would be to track the progress of changes to this app discussing the reason for each one, but maybe another day, for now let simply focus on the two lines that changed here.

By the way, if you want to get back to the HEAD for the repo, just run `git checkout master`

## Bundler Setup

The first line `require 'bundler/setup'` simply runs: [this code][3] which, in turn, runs the `Bundler.setup` method. What does this do? According to the [Bundler Docs][4] it *&#8230;configures the load path so all dependencies in your Gemfile can be required&#8230;*. Essentially it means that once `Bundler.setup` has run, its then possible to require any gems as declared in the Gemfile using `require` and be certain that the correct *bundled* version of that gem will be loaded.

So `Bundler.setup` simply ensures that the load path is correctly set so that require xyz gem will load the very gem as defined in the Gemfile.lock file.

## Bundler.require

The next line does exactly what it says, it requires all the gems! In fact, this is a little known command that will require *all* the gems in the Gemfile. The alternative is to require each gem one by one, in this case &#8216;rails, &#8216;coderay&#8217; and &#8216;markaby&#8217;. However, since this is the *Smallest* Rails app, we use `Bundler.require` to require *all* gems in one swoop!

Its also possible to execute the `require` method passing in a symbol representing the group that you desire to load. For example `Bundle.require(:development)` will require *all* declared gems in the development group of your Gemfile. Its possible to pass in [multiple groups][5] separated by comma too.

## Bundler.setup and &#8216;bundle exec&#8217;

Its worth mentioning here that running &#8216;bundle exec&#8217; on the command line before starting the Smallest Rails App ensures that the load path is setup correctly (essentially the same as `require "bundler/setup"`) and thus means that its possible to run the Smallest Rails App without the `require "bundler/setup"` line. I have commented on this line of code to the author of the Smallest Rails App and he has since removed this line from the application!

## Conclusion

The Smallest Rails app will still probably be able to get smaller as gems and approaches change. In the meantime, for our purpose, it serves as an interesting digest of new techniques to learn about Rails and its associated gems (in this case Bundler!).

 [1]: http://www.tweetegy.com/2012/08/dissecting-thesmallestrailsapp-com-smallest-rails-app/ "Dissecting thesmallestrailsapp.com (Smallest Rails App) (Part 1)"
 [2]: https://github.com/artemave/thesmallestrailsapp.com/commit/3ad9c9c799ad0b4a90db2f08a53c8f31a2c13d67
 [3]: https://github.com/bundler/bundler/blob/master/lib/bundler/setup.rb
 [4]: http://bundler.io/bundler_setup.html
 [5]: http://bundler.io/v1.3/groups.html

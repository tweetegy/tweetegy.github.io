---
layout: post
author: jensendarren
title: Managing Bower Web Packages with rails-assets.org
permalink: /2014/12/managing-bower-web-packages-with-rails-assets-org
categories:
  Ruby, Rails, Programming
tags:
  - bundler
  - bower
  - package
  - gemfile
  - dependency management
  - assets
  - asset pipeline
  - rails
  - javascript
---

### Bower

Bower describes itself as a 'change manager for the web'. An easy way to understand Bower is that it is a tool for fetching various libraries and their dependenceis and includng them into your project.

For example if you needed to pull in jQuery into your project then you can run:

{% highlight bash %}
bower install jquery
{% endhighlight %}

So Bower has introduced a simple and managable way to bring in libriaries and their dependancies into your JavaScript projects. It certainly beats trying installing things manualy!

### Bundler

Bundler is a tool that has been around longer than Bower and essentially does the same thing as Bower but for Ruby Gems. A common way developers are introuduced to Bundler is via Rails since Rails uses Bundler by default. Developers simply add their applications Gem dependenices to the applications Gemfile and run `bundle install` on the command line.

### Combining Bower and Bundler with rails-assets.org

[Rails Assets](https://rails-assets.org/) is a great way to manage all your external assets in Rails. Note this is *not* a substitute for the Asset Pipeline! Rails Assets describes itself as the *frictionless proxy between Bundler and Bower*. It does this by automatically converting the packaged components (from Bower) into Ruby Gems that are easily droppable into the Rails asset pipeline and stay up to date.

### Rails Asset Pipeline

The Asset Pipeline was introduced in Rails 3.1 and is a fantasitic addition to the stack. If we add a JavaScript file to our `app/assets/javascripts` directory and we already have `require-tree` then this file will be automaticaly sent to the browser (via a call to `javascript_include_tag` in the layout file). The Asset Pipeline also takes care of the Production obfuscation and minification requiremetns as well as producing single .js and .css files amongst other things.

### The problem that rails-assets.org solves

One option for adding 'web packages' (that is things like JavaScript libraries, CSS libraries and so on) to our Rails application was to go hunting for the libary file itself and drop that into 'lib/assets/..' (or other) so that it is pulled into the asset pipline. The problem here was finding ways to keep these files up to date and handle dependencies correctly.

An alternative was to go hunting for a gem that is built with the intension of wraping the web package in a Rails Engine that can be installed and therefore included automatically in the asset pipleine. The problem here is that the versioning becomes quite confusing (one version for the Gem and one for the web package) and if you want to include a later version of the framework then you would probably have to fork the Gem yourself.

### The future is here with rails-assets.org !

Enter rails-assets.org - which allows developers to simply add the web package to their Gemfile and install via `bundle install` command. Very familiar, easy to setup and no hassel. Developers no longer any need to maintain separate Gems for intsalling web packages like jQuery or Bootstrap or Angular JS - just use rails-assets.org from now on!

### Getting started with rails-assets.org

To get started using the rails-assets.org 'frictionless proxy' just do the following:

Add https://rails-assets.org as a new gem source. For example the top of your Gemfile might look like this:

{% highlight ruby %}
source 'https://rubygems.org'
source 'https://rails-assets.org'

# Gems listed here ...
{% endhighlight %}

Add any Bower related web packags as you would any new Gem using the syntax `gem 'rails-assets-BOWER_PACKAGE_NAME'`. For example the following is an example Gemfile that includes Bootstrap, AngularJS and Leaflet librarries via Bower!

{% highlight ruby %}
source 'https://rubygems.org'
source 'https://rails-assets.org'

gem 'rails-assets-bootstrap'
gem 'rails-assets-angular'
gem 'rails-assets-leaflet'
{% endhighlight %}

Now you can require the BOWER-PACKAGE-NAME in your asset pipline as follows. This is how we can include Bootstrap, Angular JS and Leaflet JavaScript files in the asset pipleine:

{% highlight ruby %}
//= require_self
//= require bootstrap
//= require angular
//= require leaflet
//= require_tree .
//= require_tree shared
{% endhighlight %}

Here is an example of including CSS dependencies into your Rails Application:

{% highlight ruby %}
/*
  *= require_self
  *= require bootstrap
  *= require leaflet
  *= require_tree .
  *= require_tree shared
  */
{% endhighlight %}

Please checkout the [Rails Assets](https://rails-assets.org/) website for further details about getting started.

I hope you get a chance to try out rails-assets.org in your Rails projects and you will never go back to the old way again!

---
title: 'Legacy Rails app (well 2.3.x) with (almost) latest Rspec, Capybara &#038; Factory Girl'
author: jensendarren
layout: post
permalink: /2014/03/legacy-rails-app-well-2-3-x-almost-latest-rspec-capybara-factory-girl/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - 2.3.11
  - legacy
  - rails
  - rspec
  - spec
  - testing
---
I recently had a look at an oldish Rails project that uses Ruby version 1.8.7 and is running version 2.3.11 of Rails. One of the problems with this project was that it is completly untested! So I thought, since I prefer TDD (as it actually keeps me sane!) I would setup Rspec, Capybara and Factory Girl for this project.

My initial thoughts were that this would not be easy since it was an old version of Rails tied (currently) to Ruby 1.8.7. Fortunately, this turned out not to be the case and I found that [Rspec version 2.11.0][1] happily works!

## Why I like TDD?

Just a quick note on why I like TDD and why I say it keeps me &#8216;sane&#8217;. The main reasons are:

*   I know my stuff works
*   I can refactor with confidence
*   Its easier to develop and debug in test environment
*   I can sleep well at night!

Anyway, enough of that, lets get onto the main topic of question here&#8230;

## Gemfile

Now I can add the remaining gems directly to the Gemfile like so:

{% highlight ruby linenos %}group :test do
  gem 'rspec', '2.11.0'
  gem 'capybara', '1.1.4'
  gem 'pry'
  gem 'factory_girl'
  gem 'database_cleaner'
  gem 'shoulda-matchers'
end
{% endhighlight %}

So then all that was left was to let Bundler do the heavy lifting for finding and associating the correct versions of the Gem dependencies for all of these!

## Sample Rails 2.3.11 project

I have made a simple Rails 2.3.11 project that uses the above Gems and has some example specs to show that it does indeed work. It can be downloaded here: [rspec\_legacy\_rails][2]

You will notice that we have had to explicitly set certain gem versions like nokogiri and mime-types etc. This is mainly due to the project using ruby 1.8.7 and most of these gems complain about that and ask for ruby 1.9.2 (or higher). While it would be better to upgrade ruby, since we assume, this project does not have any tests (for now) we&#8217;ll keep the version of Ruby set as it is. However, generally, for projects like this, I would spend a week or two getting a decent amount of the features (not all) covered with specs and then upgrade Ruby then upgrade Rails.

The thing that is great about this approach so far is that we are able to use a relatively new version of our testing frameworks. This means we don&#8217;t have to rewrite specs again once we upgrade. We should be able to fix any issues associated with the upgrade, maybe do some refactoring and then we are good to go!

## Spec Helper File

This is probably the most important file for setup of the environment. What I have put into the file can probably be refactored. There are some lines I need to explain briefly too:

{% highlight ruby linenos %}# This ensures that all the gems are required before we begin. Saves writing 'require ...' for each gem!
Bundler.require(:test)
{% endhighlight %}

{% highlight ruby linenos %}# This ensures that we have access to all the url helpers in the specs.
config.include ActionController::UrlWriter
{% endhighlight %}

{% highlight ruby linenos %}# This ensures that we can use the Capybara DSL methods like 'visit' for our integration specs
config.include Capybara::DSL
{% endhighlight %}

## Binding.pry and save\_and\_open_page

Another cool thing is that both `binding.pry` and `save_and_open_page` both work in this environment.

## Conclusion

The main point of all this is that while the project was &#8216;legacy&#8217; Rails, using an &#8216;ancient&#8217; version of Ruby and (worst of all) completely untested, I was able to set myself up with a comfortable test environment which (almost) resembled my environment for some more modern Rails (think version 4.x) applications that I am involved in. This allowed me to get on and add features, refactor and deploy with confidence (and sleep at night too!).

One other thing to note is that I have ONLY tried out Model and Integration (or Feature) specs in this app. This is mainly becuase these are the main set of spec types that I usually work with. Having said that, the Controller specs would be useful if we need to work with legacy Rails application that expose a RESTful API for a single page app, for example. Unfortunately, I can confirm that Controller specs do not appear to work in this setup, but feel free to have a go!

 [1]: https://rubygems.org/gems/rspec/versions/2.11.0
 [2]: https://github.com/tweetegy/rspec_legacy_rails

---
title: Using MongoDB as a logging database with a Rails 3 application
author: jensendarren
layout: post
permalink: /2011/01/using-mongodb-as-a-logging-database-with-a-rails-3-application/
seo_follow:
  - 'false'
image:
  -
categories:
  - Databases
  - nosql
tags:
  - mongodb
  - mongomapper
  - rails3
  - ruby
---
Here is a great article on the MongoDB site that explains how to [setup Rails 3 to work with MongoDB with very little effort!][1]

Essentially this means doing the following things.

Create a new Rails 3 application issuing the skip-active-record switch

{% highlight bash %}
rails new my_app --skip-active-record
{% endhighlight %}

Add the following line to include mongo_mapper to your gemfile

{% highlight ruby %}
gem "mongo_mapper"
{% endhighlight %}

Now run the bundle installer

{% highlight bash %}
bundle install
{% endhighlight %}

Finally, add this initializer to your Rails 3 project to connect to your MongoDB app:

{% highlight ruby linenos %}
MongoMapper.connection = Mongo::Connection.new('localhost', 27017)
MongoMapper.database = "#myapp-#{Rails.env}"

if defined?(PhusionPassenger)
 PhusionPassenger.on_event(:starting_worker_process) do |forked|
   MongoMapper.connection.connect_to_master if forked
 end
end
{% endhighlight %}

Now it's time to create our dummy action:

{% highlight bash %}
rails g controller DummyAction simulate
{% endhighlight %}

Creating models that use MongoMapper is very easy. This is a good thing because, at the time of wrting, Rails 3 generators for MongoMapper are not included, so you might end up trying what I did and getting **&#8220;No value provided for required options &#8216;&#8211;orm&#8217;&#8221;** error and then trying all combinations under the sun for the orm switch to get the generator to work! If you want to use generators then try [Rails 3 generators][2], which is a project that simply has: *&#8220;Rails 3 compatible generators for gems that don’t have them yet&#8221;*

So I created my own models by hand because it is so easy to do. A typical MongoMapper model might look like this WebAppVisitorLog class of mine:

{% highlight ruby linenos %}
class WebAppVisitorLog
  include MongoMapper::Document

  key :session_id, Integer
  key :user_id, Integer
end
{% endhighlight %}

Finally, what to do with the action? Well push a log entry to MongoDB like so:

{% highlight ruby linenos %}
class DummyActionController < ApplicationController
  def simulate
    #log some fake data in Mongo DB
    @log = WebAppVisitorLog.create(
    {
      :session_id => 1,
      :user_id => 1,
      :updated_at => Time.now
    })
    @log.save()
  end
end
{% endhighlight %}

Be sure to add this to your config.rb file also:

{% highlight ruby linenos %}
get "dummy_action/simulate"
{% endhighlight %}

 [1]: http://www.mongodb.org/display/DOCS/Rails+3+-+Getting+Started
 [2]: https://github.com/indirect/rails3-generators/

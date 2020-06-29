---
title: Install MongoDB and load dummy logging data using MongoMapper
author: jensendarren
layout: post
permalink: /2011/01/install-mongodb-and-load-dummy-logging-data-using-mongomapper/
image:
  -
seo_follow:
  - 'false'
categories:
  - Databases
  - nosql
  - Ruby Server Programming
tags:
  - mongodb
  - mongomapper
  - nosql
  - ruby
---
It turns out to be [very easy to install Mongo DB on Ubuntu][1], so I will not go into detail too much here. If you follow these instructions then MongoDB will start after the installation completes and you can try it out at the console as follows (Note: The first command will fire up the Mongo DB shell and the second will list all the databases):

{% highlight bash %}
mongo
show dbs
{% endhighlight %}

The shell is probably the best way to get started with MongoDB. [Here are some more details on using the MongoDB shell][2]. Or just type *help* in the console!

Another useful trick to note is that it&#8217;s possible to get some administrative details about the server by pointing your browser to the following location: <http://localhost:28017/>. This displays details like version number, database stats, namespace information etc.

**Loading MongoDB with test data.**

There are three ways to insert data directly into MongoDB. There is the standard **insert** command which inserts one document at a time, there is the **batch insert** which can be used to insert multiple documents at a time (this is more efficient than inserting one at a time), or there is the **mongoimport** command line utility for running a raw data import from a relational database like MySQL. Many of these I will cover in future posts.

There also are indirect ways to load data namely via database language adapters like MongoMapper for Ruby. Here is an example of generating random data and loading it into MongoDB:

{% highlight ruby linenos %}require 'rubygems'
require 'mongo'
require 'mongo_mapper'
require 'faker'
require 'WebAppVisitorLog'

#Create a couple of types of documents (see the classes)
MongoMapper.database = "mongodb-logging"

WebAppVisitorLog.destroy_all

10000.times {|n|
  user_id = (1..1000).to_a.rand
  session_id = (1..10000).to_a.rand
  updated_at = (1..35).to_a.rand.days.ago

  @log = WebAppVisitorLog.create(
            {
              :session_id => session_id,
              :user_id => user_id,
              :updated_at => updated_at
            })
  @log.save()
}
{% endhighlight %}

Here is the code for the WebAppVisitorLog class:

{% highlight ruby linenos %}class WebAppVisitorLog
  include MongoMapper::Document

  key :session_id, Integer
  key :user_id, Integer
end
{% endhighlight %}

 [1]: http://www.mongodb.org/display/DOCS/Ubuntu+and+Debian+packages
 [2]: http://www.mongodb.org/display/DOCS/Tutorial

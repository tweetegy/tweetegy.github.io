---
title: Using Value Objects in ActiveRecord
author: jensendarren
layout: post
permalink: /2011/07/using-value-objects-in-activerecord/
image:
  -
seo_follow:
  - 'false'
categories:
  - Ruby Server Programming
tags:
  - activerecord
  - ruby
  - valueobject
---
I wanted to find out more about using Value Objects in ActiveRecord. Value objects are conserned with the value of their attributes over other identifiers (as opposed to Entitiy Objects which are concerned mainly about a unique ID). I decided to create a stand alone Active Record project (i.e. without using Rails). This became a sort of mini exercise in itself.

## Structuring a standalone ActiveRecord project

I like structure, so the setup I settled with is this:

/db
/models
/config
run.rb

Inside config I placed two files a config.rb and database.yml file:

#### config.rb

{% highlight ruby linenos %}
require 'rubygems'
require 'active_record'
require 'yaml'
require 'logger'

DBCONFIG = YAML::load(File.open('config/database.yml'))
ActiveRecord::Base.establish_connection(DBCONFIG)
ActiveRecord::Base.logger = Logger.new(File.open('db/database.log', 'a'))
{% endhighlight %}

The database.yml file simply contains the details for my database connection. Inside my schema.rb I use `ActiveRecord::Migration` to up (create) and down (drop) my table in the database. All the details can be found in the sample project on [Bitbucket][1].

## Structuring Value Objects

The interesting part when concerning Value Objects are the two classes under the models directory person.rb and address.rb (shown below). Notice that since we want address data to be stored as a Value Object we do not inherit from `ActiveRecord::Base`. Instead, address is a plain Ruby object and is used in Person by using the *composed_of* class method. This allows for the attributes of the Value Object to be stored together in the database and *composed_of* allows us to use these together as one object.

#### person.rb

{% highlight ruby linenos %}
require 'models/address'

class Person < ActiveRecord::Base
  composed_of :address, :mapping => [%w(address_city city), %w(address_county county)]
end
{% endhighlight %}

#### address.rb

{% highlight ruby linenos %}
class Address
  attr_reader :city, :county
  def initialize(city, county)
    @city, @county = city, county
  end
  def ==(other_address)
    city == other_address.city && county == other_address.county
  end
end
{% endhighlight %}

## Using Value Objects

So how to use Value Objects? Below is a sample application run.rb that uses the above models and creates four records in the database and searches for using a Value Object for records that have a particular address. The output from this applicaition is not surprisingly: Darren, Michael, proving that it is possible to pass in a new instance of a Value Object (in this case Address) and ActiveRecord will break that down and search for the address in the database for us.

#### run.rb

{% highlight ruby linenos %}
require 'config/config'
require 'models/person'

p1 = Person.create(:name => "Darren", :address_city => "Eastbourne", :address_county => "Sussex")
p2 = Person.create(:name => "Paul", :address_city => "Nottingham", :address_county => "Nottinghamshire")
p3 = Person.create(:name => "Simon", :address_city => "Dartford", :address_county => "Kent")
p4 = Person.create(:name => "Michael", :address_city => "Eastbourne", :address_county => "Sussex")

people_in_eastbourne_sussex = Person.find_all_by_address(Address.new("Eastbourne", "Sussex"))
puts people_in_eastbourne_sussex.map(&:name)
{% endhighlight %}

The SQL that is generated for find\_all\_by_address is not surprising:

{% highlight sql %}SELECT "people".* FROM "people"
WHERE "people"."address_city" = 'Eastbourne'
AND "people"."address_county" = 'Sussex'
{% endhighlight %}

Another thing to note is that it is possible to reference either the Value Object via *person.address* or the attributes separately via *person.address_city* and *person.address_county*.

 [1]: https://bitbucket.org/darren_jensen/tweetegy-value-objects/

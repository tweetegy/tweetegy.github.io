---
title: Using postgres hstore with Rails / Active Record
author: jensendarren
layout: post
permalink: /2014/06/using-postgres-hstore-rails-active-record/
image:
  -
categories:
  - Databases
  - nosql
  - Rails Appplication Programming
tags:
  - activerecord
  - hstore
  - nosql
  - postgres
  - rails
---
One good solution for implementing key / value persistence to an Active Record model is to use a postgres hstore column. This gives you the ability to store any key / value pair (as strings) directly on the model and manipulate them as first class attributes.

In this post, I&#8217;ll demonstrate how to set up a Rails application to use hstore and I&#8217;ll also show some of the pitfalls and limitations of this technique.

## Enable your app to use hstore

Once you have postgres installed, the only requirement is to enable hstore in the database. This can be done by running the following command in postgres:

{% highlight sql %}CREATE EXTENSION hstore;
{% endhighlight %}

This can go in a migration too, of course, which is recommended. Just create the migration file in the usual way and add the above statement to the `up` method of the migration and add the `DROP EXTENSION` equivalent to the `down` method of the migration. Here is an example:

{% highlight ruby linenos %}
class SetupHstore < ActiveRecord::Migration
  def self.up
    execute "CREATE EXTENSION IF NOT EXISTS hstore"
  end

  def self.down
    execute "DROP EXTENSION IF EXISTS hstore"
  end
end
{% endhighlight %}

Now we can create a migration for our model that we want to use hstore on. A typical example is where we might have a single 'thing' (lets say Product) that have some common attributes (like name, price) but also have different properties depending on the actual product (like 'dpi' specifically for a camera but not for shoe for example!).

So lets look at an example migration for Product that contains a properties hstore column:

{% highlight ruby linenos %}
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.string :description
      t.float :price
      t.hstore :properties

      t.timestamps
    end
  end
end
{% endhighlight %}

As you can see from `t.hstore :properties`; hstore is treated in the migration a first class database type. Run the migrations and this will enable the hstore extension in your database and add the products table with the hstore column.

## Playing around with hstore in the console

Lets start with getting a feel for hstore in the console in order to understand what it can do for us and what its limitations are. The following code will create a new Product item leaving properties nil.

{% highlight ruby linenos %}Product.create(name: "Cannon D30", description: "A great digital SLR", price: 399.99)
Product.first.properties # => nil
{% endhighlight %}

Lets set some key / value items on properties. Lets add the following (<a href="http://en.wikipedia.org/wiki/Comparison_of_Canon_EOS_digital_cameras" title="Comparison of Canon EOS digital cameras" target="_blank"></a>):

Sensor Format: APS-C CMOS
Megapixels: 3.1
Min ISO: 100
Max ISO: 1600

Here is the code to update the properties on this product instance:

{% highlight ruby linenos %}camera_properties = {sensor_format: "APS-C CMOS", megapixels: 3.1, min_iso: 100, max_iso: 1600}
p = Product.first
p.update(properties: camera_properties)
{% endhighlight %}

Now, if we inspect the properties attribute on the product the output is:

{% highlight ruby linenos %}p.properties #=> {:sensor_format=>"APS-C CMOS", :megapixels=>3.1, :min_iso=>100, :max_iso=>1600}
{% endhighlight %}

Interestingly, if we call `reload` we see a slightly different output, namely that both the keys and values are now all strings:

{% highlight ruby linenos %}p.reload.properties #=> {"max_iso"=>"1600", "min_iso"=>"100", "megapixels"=>"3.1", "sensor_format"=>"APS-C CMOS"}
{% endhighlight %}

This is because the postgres database stores all hstore key / values as strings regardless of the type in Ruby. Inspecting the database row shows this:

{% highlight sql %}select name, properties from products;
    name     |                                       properties
-------------+-----------------------------------------------------------------------------------------
 Cannon D30 | "max_iso"=>"1600", "min_iso"=>"100", "megapixels"=>"3.1", "sensor_format"=>"APS-C CMOS"
(1 row)
{% endhighlight %}

## Adding more properties to an existing model

The cool thing about using hstore is that we can add new attributes to the column very easily. For example we might decide that we want to include information about the storage format of the camera. We can easily do this by adding this key / value to the properties. The important thing to note is that we must merge the new properties into the existing hash:

{% highlight ruby linenos %}p.update(properties: p.properties.merge({storage: "CF"}))
{% endhighlight %}

## Accessing the properties in the model

In order to access the hstore properties directly on the model we could write our own custom accessor methods like so:

{% highlight ruby linenos %}def storage
  self.properties["storage"]
end
{% endhighlight %}

A neater approach is to use the `store_accessor` helper that creates these methods on the fly. This is how we can use it to create a storage method similar to the one above:

{% highlight ruby linenos %}store_accessor :properties, :storage
{% endhighlight %}

The result is the same in that we can access storage as an attribute directly on the Product instance. However, the `store_accessor` method gives us the ability to validate and set the property too! The Product class looks like this now:

{% highlight ruby linenos %}class Product &lt; ActiveRecord::Base
  store_accessor :properties, :storage
  validates_presence_of :storage
end
{% endhighlight %}

In the console, we can see that product behaves as expected:

{% highlight ruby linenos %}p = Product.first
p.storage = nil
p.valid? # => false
p.errors.messages[:storage] # => ["can't be blank"]
{% endhighlight %}

## Storing different data types

As mentioned, the hstore only stores keys / values as strings. However, since we can validate properties using standard Rails validations its possible to ensure a certain type is pushed to the hstore before it is saved in postgres as a string. That way we can always convert the type back when we want to use it. Here is an example where we might want to validate and store a number and then convert it back again when reading the value.

Its possible to add this as a validation directly to the model, like so:

{% highlight ruby linenos %}class Product &lt; ActiveRecord::Base
  store_accessor :properties, :cost
  validates_numericality_of :cost
end
{% endhighlight %}

Once that is in place in the class, you can try out the functionality in the console as follows:

{% highlight ruby linenos %}p.cost = "this should not be allowed!"
p.valid?
p.errors.messages[:cost] # => ["is not a number"]
{% endhighlight %}

## Storing nested hashes

Since hstore stores keys and values as strings it means that any nested hashes will also be converted to a string. Here is an example. Given the following nested hash:

{% highlight ruby linenos %}nested_properties_hash = {storage: "5GB", cost: "499", warranty: {first: "OK", second: "OK"}}
{% endhighlight %}

Setting this to the Product.properties is valid and so can be saved. However, note that the keys are converted to strings and that the nested part of the hash is escaped:

{% highlight ruby linenos %}p.properties = nested_properties_hash
p.save
p.reload.properties # => {"cost"=>"499", "storage"=>"5GB", "warranty"=>"{\"first\"=>\"OK\", \"second\"=>\"OK\"}"}

{% endhighlight %}

Notice that the top level is *still* a Hash but all the nested levels are now Strings (in our case 'warranty'). Its possible to convert all the nested levels to Hashes again using `eval` but I **do not recommend this** as your code can get really messy:

{% highlight ruby linenos %}eval(p.properties["warranty"]) # => {"first"=>"OK", "second"=>"OK"}
{% endhighlight %}

## Conclusion

Using postgres hstore with Rails is really useful for flat (single level) key/value string store. Its possible to ensure a particular type by using validation and then convert that back after a read from the database. If there is a requirement to store more complex data structure, such as a nested Hash, then it would be better to use another storage mechanism such as a document base 'nosql' database.

As always, here is [an example application][1]

 [1]: https://github.com/tweetegy/hstore_example

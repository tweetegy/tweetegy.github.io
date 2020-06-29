---
title: Setting join table attribute :has_many :through association in Rails ActiveRecord
author: jensendarren
layout: post
permalink: /2011/02/setting-join-table-attribute-has_many-through-association-in-rails-activerecord/
image:
  -
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
tags:
  - associations
  - has_many
  - rails
  - ruby
  - through
---
I needed to set an attribute on a join table or in ActiveRecord, the* :through* model as an atomic save action. Let's set the scene: I have a *Product *and a *User *which has a many-to-many relationship since a *Product *can have many *Users *and a User can have many *Products*.

Typically, in a database we might have a table *product*, user and *product_user*. In Rails I created an association class called *Collaborators *since it fit my domain better. Additionally the *Collaborators *class has one attribute *is_admin* which indicates the *User *(or *Collaborator *actually) is an administrator for the particular *Product*.

One of the business rules in the application was that if a *User *creates a new *Product *then they are automatically an administrator for that *Product*. My dilemma was how to create the new *Product*, associate it correctly with the *User *\*and\* set* is_admin* to true all in one nice and neat command?

The answer, it turns out, is straightforward and elegant. I am fairly new to Rails myself, so I understand if this might appear a little overwhelming if you are also just starting out. I have included a set of useful references at the end of this post to help you on your way.

{% highlight ruby linenos %}
class Product < ActiveRecord::Base
  has_many :collaborators
  has_many :users, :through => :collaborators
end
{% endhighlight %}

{% highlight ruby linenos %}
class User < ActiveRecord::Base
  has_many :collaborators
  has_many :products, :through => :collaborators
end
{% endhighlight %}

{% highlight ruby linenos %}
class Collaborator < ActiveRecord::Base
  belongs_to :product
  belongs_to :user
end
{% endhighlight %}

Once you have the above ActiveRecord models setup all you need to do is execute the following code to save a new *Product*, associate a *User *as a *Collaborator *and set them as an administrator for the *Product*.

{% highlight ruby linenos %}
#assume we have the product and current_user instance already set
product.save && product.collaborators.create(:user => current_user, :is_admin => true)
{% endhighlight %}

**Useful Resources and Docs for ActiveRecord Associations**
[Has Many Through Association][1]
[ActiveRecord::Base Documentation][2]

 [1]: http://guides.rubyonrails.org/association_basics.html#the-has_many-through-association
 [2]: http://ar.rubyonrails.org/classes/ActiveRecord/Base.html

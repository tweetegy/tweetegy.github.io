---
title: Migrate a PHP application to Rails 3
author: jensendarren
layout: post
permalink: /2011/05/migrate-a-php-application-to-rails-3/
image:
  -
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
tags:
  - migration
  - php
  - rails
  - tips
---
I recently migrated a basic PHP website to Rails 3. The job went smoothly and this is my first Rails 3 app so I thought I would write about my experiences.

## Getting Started

In this situation I would start directly at database level. Get the database ready so that you can create simple ActiveRecord classes without any rule bending. Here are some tips:

*   Make sure all your table names are pluralized
*   Make sure all your tables contain an `id` column as well as `created_at` and `updated_at` columns to adhere to Rails convention.
*   Update the parent level tables first, create the models and quickly test them out in the console one-by-one
*   Move on to child tables / associations and test these too in the console

**Note:**

Because you are building this inside out so to speak (i.e. not using generators) you might come across issues like wanting to use reserved words for your models. in my case I needed to name a model Class (as in a taught class) but had to rename to Klass and use set\_table\_name as follows:

{% highlight ruby linenos %}
class Klass < ActiveRecord::Base
  set_table_name "classes"
end
{% endhighlight %}

## Date Formatting in Rails 3

Simply create a file under `config/initializers/` directory called `time_formats.rb` and include formats that you might be needing throughout your application. in my case I wanted dates to be output as Full day, Day number, Month, Year e.g. Monday, 5th April 2011

{% highlight ruby linenos %}
Date::DATE_FORMATS[:schedule_date] = "%A %d %B %Y"
{% endhighlight %}

## Rendering Collections with Partials in Rails 3

Something else that is quite useful is rendering partials with collections. By simply passing a collection to the render method, Rails will automatically render a partial with the singular name from the models view directory.

For example, I have a list of courses under each category that I want to render, so by passing category.courses the view views/course/_course.html.erb is rendered for each course item in the collection. Marvelous!

{% highlight erb linenos %}
<%= render category.courses %>
{% endhighlight %}

## Select drop down lists in Rails 3

Making select drop downs is amazingly clear and simple also. I made use of `options_from_collection_for_select` which is a very neat and powerful way to generate a set of options from an existing collection. Used in conjunction with `select_tag` renders our completed drop down list of options!

{% highlight erb linenos %}
<%= select(:enrollment, :class_id, options_from_collection_for_select(@klasses, :id, :name)) %>
{% endhighlight %}

## Showing users error notifications in Rails 3

For some reason, the form builder helpers `error_messages` have been removed from Rails 3. I presume it is to allow the developer more control over rendering any validation errors. So if you want to output errors, use the following example to guide you:

{% highlight erb linenos %}
<%= form_for(@enrollment) do |f| %>
<% if @enrollment.errors.any? %>
  <ol>
    <% @enrollment.errors.full_messages.each do |error_msg| %>
    <li>
      <%= error_msg %>
    </li>
    <% end %>
  </ol>
<% end %>
{% endhighlight %}

## SEO Friendly urls

SEO Friendly urls are really easy to setup so put this in yer model and smoke it! The `to_param` method is used by the url helpers so by overriding in this way autmatically generates the url you choose with no further fuss!

{% highlight ruby linenos %}
def to_param
  "#{id}-#{title.parameterize}"
end
{% endhighlight %}

## Sitemap XML file generation example in Rails 3

Sitemap XML file generation, I recommend: [ Sitemap Generator for Rails][1]

Note to get this working you need to add the following include to your rake task

{% highlight ruby linenos %}
include Rails.application.routes.url_helpers
{% endhighlight %}

...and also the following configuration change to your application.rb config

{% highlight ruby linenos %}
config.autoload_paths += %W(#{config.root}/lib)
{% endhighlight %}

## Conclusion

I am very happy that I moved my site to Rails 3 from PHP. It's much cleaner now, easier to maintain and fits in with my own personal desire to focus on building applications with Rails!

 [1]: http://blog.dynamic50.com/2010/01/27/sitemap-generator-for-ruby-on-rails-applications/

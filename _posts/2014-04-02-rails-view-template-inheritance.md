---
title: Rails View Template Inheritance
author: jensendarren
layout: post
permalink: /2014/04/rails-view-template-inheritance/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - inheritance
  - rails
  - template inheritance
  - templates
  - view inheritance
  - views
---
View Inheritance has been in Rails since version 3.1. In this post, I cover a Rails 4 example and as far as I know what is covered here in this blog and in the example application is compatible with the earliest version of View Inheritance in Rails 3.x.

## View inheritance maps to Controller inheritance

In a Rails application, one will generally inherit most of the applications controllers from `ApplicationController`. Lets say a very basic course management application will have Category, Course and Tutor controllers then, by default, each controller will inherit from ApplicationController. Rails will also expect a corresponding view folder for each controller like so:

{% highlight bash %}views ->
  categories #folder for CategoriesController
  courses #folder for CoursesController
  tutors #folder for TutorsController
{% endhighlight %}

Passing in actions to the Rails Controller generator will also create corresponding view templates for those stated actions under the corresponding view directory:

{% highlight ruby linenos %}
rails g controller categories index show

# This creates the following Controller:

class CategoriesController < ApplicationController
  def index
  end

  def show
  end
end

# ... and it will also create index.html.erb
# and show.html.erb under views -> categories folder
# (as well as helpers, assets, specs etc)
{% endhighlight %}

The standard behaviour for this application is to visit the index for a resource, like `/categories` which will flow through the application eventually calling the `index` method on `CategoriesController` and by Rails conventions, it will render the &#8216;app/views/categories/index.html.erb&#8217; template.

## Delete the categories/index.html.erb

Go on! Delete the index.html.erb template under views/categories folder! Don&#8217;t be scared. In fact you will discover something quite interesting in the error message that is returnd by Rails that gives us a clue to how view template inheritance works. I&#8217;ve already tried it, here is the part of the error message that spills the view template inheritance beans:

*Missing template categories/index, application/index with &#8230;# blah blah*

Notice that Rails looks in **two places** for our template, both in &#8216;categories/index&#8217; (as we expect) *and* &#8216;application/index&#8217;. What this means is that *if* we place an index.html.erb file in &#8216;views/application&#8217; directory (keeping the index file deleted under &#8216;categories&#8217;) then Rails will render the index.html.erb file from &#8216;views/application&#8217; folder when we visit `/categories`!

## Real World Example

This approach can be useful for exactly what it says on the tin &#8216;inheritance&#8217;. If we wanted, we could place some common view output inside a template that is under the &#8216;views/application&#8217; directory. A common example might be a menu for your application that is common for most of the application but maybe slightly different for other parts.

Here is an example where we use partials to achieve this:

{% highlight bash %}views ->
  application ->
    _menu.html.erb
  categories ->
    index.html.erb
  courses ->
    index.html.erb
  tutors ->
    _menu.html.erb
    index.html.erb
{% endhighlight %}

In the above example Rails application views folder structure we have each folder with an index page and each index page renders out the menu partial like so:

{% highlight ruby linenos %}render 'menu'
{% endhighlight %}

In categories and courses, the menu partial is not present in the categories or courses views directory so Rails will use view inheritance and look up the tree. **Here is the key**: because `CategoriesController` and `CoursesController` *inherit* from `ApplicationController` the next logical folder that Rails will search for the menu partial is under &#8216;views/application&#8217;.

What we end up with is an application which renders the same menu partial on every page except where it is overridden, in this case under tutors (maybe to add a specific apply for teaching role link or something!).

Here are the two menu partial templates, one under &#8216;views/application/&#8217; and the other under &#8216;views/tutors/&#8217;

{% highlight ruby linenos %}
# In 'views/application/_menu.html.erb'
<ul>
  <li>
    <%= link_to "Home", root_path %>
  </li>

  <li>
    <%= link_to "Categories", categories_path %>
  </li>

  <li>
    <%= link_to "Courses", courses_path %>
  </li>

  <li>
    <%= link_to "Tutors", tutors_path %>
  </li>
</ul>
{% endhighlight %}

{% highlight ruby linenos %}# In 'views/tutors/_menu.html.erb' (note extra menu item at the bottom)
<ul>
  <li>
    <%= link_to "Home", root_path %>
  </li>

  <li>
    <%= link_to "Categories", categories_path %>
  </li>

  <li>
    <%= link_to "Courses", courses_path %>
  </li>

  <li>
    <%= link_to "Tutors", tutors_path %>
  </li>

  <li>
    <b><a href='#'>Apply for teaching position!</a></b>
  </li>

</ul>
{% endhighlight %}

## Removing the duplication

In the example above we have duplicated the first set of menu items in the &#8216;tutors/_menu.html.erb&#8217; file. Ideally, it would be nice to avoid that duplication and just add the extra menu items that we need. The trick is to use a placeholder for where we want the extra items to appear (in this case at the end). We also need to provide an empty placeholder template in the &#8216;views/application&#8217; directory so that Rails does not throw a missing template error.

What we end up with is the following in our templates:

{% highlight ruby linenos %}# In 'views/application/_menu.html.erb'
<ul>
  <li>
    <%= link_to "Home", root_path %>
  </li>

  <li>
    <%= link_to "Categories", categories_path %>
  </li>

  <li>
    <%= link_to "Courses", courses_path %>
  </li>

  <li>
    <%= link_to "Tutors", tutors_path %>
  </li>
  <%= render 'appended_menu' %>
  # './application/_appended_menu.html.erb' is an empty file
</ul>
{% endhighlight %}

Now we can delete the menu partial under &#8216;views/tutors&#8217; and instead add the &#8216;appended_menu&#8217; partial as follows:

{% highlight ruby linenos %}# In 'views/tutors/_appended_menu.html.erb'
<li>
  <b><a href='#'>Apply for teaching position!</a></b>
</li>
{% endhighlight %}

So now thanks to Rails view inheritance the application will run without error and for the tutors page the application will render the tutors version of the appended_menu partial thus adding our additional link to the page all while keeping the views dry! Sweet!

As you can probably tell this approach only works if we need to *append* something to the menu. What happens if we want to put this link *before* the tutors link? We have a few options:

*   Simply move the replaceable partial placeholder before the tutors link in the application menu partial
*   Keep the &#8216;append_menu&#8217; partial where it is (for appending menu items) and make a new partial specifically before tutors link
*   If you find you need a lot of custom template overriding like this, use the [deface gem][1] (or similar).

## Going further up the tree!

Here is quick example that demonstrates a namespaced controller under the namespace Admin. So we create a new Admin::TutorsController which we want to inherit from Admin::BaseController using the following generator command:

{% highlight bash %}rails g controller admin/base
rails g controller admin/tutors index
{% endhighlight %}

We end up with a slight change to our views folder structure, namely the following has been added:

{% highlight bash %}
views ->
  admin ->
    base ->
  tutors ->
    index.html.erb
{% endhighlight %}

We also manually inherit the `Admin::TutorsController` from `Admin::BaseController` like so:

{% highlight ruby linenos %}
module Admin
  class TutorsController < BaseController
    def index
    end
  end
end
{% endhighlight %}

Now we can still render the menu partial from the admin/tutors/index template and Rails will fallback to the partial under 'views/application' because `Admin::TutorsController` inherits from `Admin::BaseController` which, in turn, inherits from `ApplicationController`!

Now comes the inheritance magic! If we place an 'appended_menu' partial under 'views/admin/base' then this partial will be rendered to add additional menu items but only when the user visits any path under the admin namespace. So its possible to place administrator links here like 'manage tutors' or whatever and these will only be rendered when visiting 'admin/tutors/' path in the browser and not '/tutors'. Try out the sample app (link below) for a demonstration of this.

## Conclusion

View inheritance can be really powerful and useful for your application. Its a nice way to dry up views and keep partials and content in their place. If you need to perform more complex view manipulations you can always use the [deface gem][1].

An example application to demonstrate this can be downloaded from here: [View Template Inheritance][2]

 [1]: https://github.com/spree/deface
 [2]: https://github.com/tweetegy/view_template_inheritance/

---
title: Create Nested Polymorphic Comments with Rails and Ancestry Gem
author: jensendarren
layout: post
permalink: /2013/04/create-nested-comments-in-rails-using-ancestry-gem/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - activerecord
  - acts_like_tree
  - ancestry
  - nested
  - ploymorphic
  - rails
  - rails 3
  - ruby
  - tree
---
I know the post title sounds a little complex and a real mouth full but, hey, its just a commenting solution that can be applied to any resource in a Rails application. Additionally, comments can be commented on (hence &#8216;nested polymorphic comments&#8217;).

By the way, this post comes with a [full working Rails Application example][1] available on Github. The code is not *exactly* the same as shown below but the application should work if you follow the README!

So, therefore, in this post I am going to build an application that uses the Ancestry Gem to enable a hand rolled nested commenting system! You can think of this as an example that combines the techniques demonstrated in [RailsCasts episode 154][2] &#8211; polymorphic associations and [RailsCasts episode 262][3] &#8211; trees with ancestry.

## Installation

First up, install the excellent [Ancestry Gem][4]. I am using Rails 3 so this is what I will demonstrate here. As always, its pretty straightforward to install and setup. Simply,

Add this to your Gemfile (and, of course, run &#8220;bundle&#8221; to install it!)

{% highlight ruby linenos %}gem 'ancestry'
{% endhighlight %}

Add the ancestry column to any tables that require it. In my application, I&#8217;ll add it to comments as part of the &#8220;change&#8221; migration since this is a brand new application.

Here is how my migration looks after manually adding the index (obviously run rake db:migrate to update your database with these changes). Note that I add `commentable_id` and `commentable_type` so that comments is polymorphic and can be added as a `has_many` association to any model that requires comments.

{% highlight ruby linenos %}
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :content
      t.integer :commentable_id
      t.string :commentable_type
      t.string :ancestry

      t.timestamps
    end

    add_index :comments, :ancestry
  end
end
{% endhighlight %}

Add the `has_ancestry` call to your model, like so:

{% highlight ruby linenos %}
class Comment < ActiveRecord::Base
  has_ancestry
end
{% endhighlight %}

Additionally, add `has_comments` to any model that you want to make 'commentable'. Let's make this application something related to movies (as I am real fan!). In this case I'll make the `Movie` model commentable by adding a `has_many` association to it like so:

{% highlight ruby linenos %}
class Movie < ActiveRecord::Base
  has_many :comments, :as => :commentable, :dependent => :destroy
end
{% endhighlight %}

Thats it for the setup!

## Rendering out some nested comments

Since the main benefit of using the Ancestry gem is how it stores data in a tree like structure, we'll get straight on to displaying our comments as a nested tree.

Firstly create this helper method. This solution was inspired from the following [RailsCast][3] on this subject:

{% highlight ruby linenos %}
module CommentsHelper
  def nested_comments(comments)
    comments.map do |comment, sub_comments|
      content_tag(:div, render(comment), :class => "media")
    end.join.html_safe
  end
end
{% endhighlight %}

Next, call this helper method from your *show* view to render the nested comments!

{% highlight ruby linenos %}= nested_comments @comments
{% endhighlight %}

The actual comment partial (which will live in `views/comments/_comment.html.haml`) looks like this (note the recursive call to `nested_comments` in the last line!).

{% highlight ruby linenos %}
.media-body
  %h4.media-heading=comment.user.name
  %i= comment.created_at.strftime('%b %d, %Y at %H:%M')
  %p= simple_format comment.content
  = nested_comments comment.children
{% endhighlight %}

## Adding a new comment / thread

Now lets look at how to create a new thread. What we need is to render at the bottom of our show view, right after the `nested_comments` call, a form to start a new comment thread. We'll render a shared partial in this case since we will want to use the same form for any thing that is commentable:

{% highlight ruby linenos %}
#In the show view
= render "comments/form"

# In comments/_form partial
= form_for [@commentable, @comment] do |f|
  = f.hidden_field :parent_id
  %p
    = f.label :content, "New comment"
    %br
    = f.text_area :content, :rows => 4
  %p
    = f.submit "Post Comment"
{% endhighlight %}

You might now be wondering where are the instance variables `@commentable` and `@comment` being set? Well we could set these directly in the `show` action of the `MoviesController`, however, that would quickly lead to code duplication when we want to make another model commentable. How about we make a module and include that into our controller instead? That way any model that is commentable, we only need to include the module into the corresponding controller. Here is the code for the `Commentable` module:

{% highlight ruby linenos %}
require 'active_support/concern'

module FilmFan::Commentable
  extend ActiveSupport::Concern

  included do
    before_filter :comments, only => [:show]
  end

  def comments
    @commentable = find_commentable
    @comments = @commentable.comments.arrange(:order => :created_at)
    @comment = Comment.new
  end

  private

  def find_commentable
    return params[:controller].singularize.classify.constantize.find(params[:id])
  end

end

# In MoviesController
class MoviesController < ApplicationController
  include FilmFan::Commentable
end
{% endhighlight %}

Thats nice as it cleans up our Controllers show actions. However, there is still an issue! The form partial `form_for [@commentable, @comment]` declaration means that we need to post to a *generic* CommentsController each time - not the actual controller that is "commetnable". This means we need to create this controller and provide a create method in it.

{% highlight ruby linenos %}
class CommentsController < ApplicationController
  def create
    @commentable = find_commentable
    @comment = @commentable.comments.build(params[:comment])
    if @comment.save
      flash[:notice] = "Successfully created comment."
      redirect_to @commentable
    else
      flash[:error] = "Error adding comment."
    end
  end

  private
  def find_commentable
    params.each do |name, value|
      if name =~ /(.+)_id$/
        return $1.classify.constantize.find(value)
      end
    end
    nil
  end
end
{% endhighlight %}

Note the `find_commentable` method courtasy of [RailsCast episode 262][3].

## Adding nested comments

Now lets combine the 'polymorphic' with the 'ancestor' and thus add the functionality to create a comment of a comment! The first thing to do is add a 'reply' link in our comment partial. This can go just above the recursive call to `nested_comments` so that we get this link rendered just after each comment. Note the use of the helper method `new_polymorphic_path` and that we still use the `@commentable` instance but in this case we pass an instance of Comment.new so that link will call the `CommentController#new` action. Note also that we are passing in a `parent_id` which the Ancestry gem requires for building the tree.

{% highlight ruby linenos %}
# start of partial ommitted....
  .actions
    = link_to "Reply", new_polymorphic_path([@commentable, Comment.new], :parent_id => comment)
  = nested_comments comment.children
{% endhighlight %}

As mentioned above, when the 'Reply' link is clicked the new action in the `CommentsController` is called. The method takes the commentable type, finds the actual instance of that using the private method `find_commentable` (shown above) and creates and new instance of Comment based on these parameters. The method looks like so:

{% highlight ruby linenos %}
class CommentsController < ApplicationController
  def new
    @parent_id = params.delete(:parent_id)
    @commentable = find_commentable
    @comment = Comment.new( :parent_id => @parent_id,
                            :commentable_id => @commentable.id,
                            :commentable_type => @commentable.class.to_s)
  end
{% endhighlight %}

The final piece of the puzzle is the form which needs to be rendered here. While it would be more desirable to render the form inline using an Ajax solution, here we will just simply render another view with the corresponding form that contains the new `Comment` object (with the correct `commentable_id`, `commentable_type` and `parent_id` all set, of course!).

{% highlight ruby linenos %}
# In comments/new.html.haml
%h1 New Comment
= render 'form'
{% endhighlight %}

Actually, the form that is rendered is the same comments form partial as shown above. Of course, in this case the @comment instance variable now represents a new *child* comment since its parent id is now set!

## Enabling another resource 'commentable'

Now lets add an `Actor` to the mix with the association `Movie <= has_and_belongs_to_many => Actors`. How would we make the Actor resource commentable? Simply follow these steps:

1.  Add "has_many :comments, :as => :commentable, :dependent => :destroy" to the Actor model
2.  Include the FilmFan::Commentable into the ActorsController
3.  Add the "nested_comments @comments" and "render "comments/form" calls to the base of the Actor show view
4.  Finally, add the nested comments resource route for actors in your routes file, just as you did with movies. </ol>
    ## Conclusion

    I hope you enjoyed this post and that you find this useful in your own projects. Here is a link to the Github project with a [full working Rails Application example][1].

 [1]: https://github.com/tweetegy/polymorphic_ancestry
 [2]: http://railscasts.com/episodes/154-polymorphic-association
 [3]: http://railscasts.com/episodes/262-trees-with-ancestry
 [4]: https://github.com/stefankroes/ancestry

---
title: 'Dissecting thesmallestrailsapp.com (Smallest Rails App: line 3)'
author: jensendarren
layout: post
permalink: /2012/12/dissecting-thesmallestrailsapp-com-smallest-rails-app-line-3/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - rack
  - rails
  - run
  - smallest
---
In this post, I will carry on from where I left off in the previous post and continue my dissection of the smallest Rails app! This takes us onto line 3 of the code (since line 2 is empty!). Line 3 looks like this:

{% highlight ruby linenos %}
run TheSmallestRailsApp ||= Class.new(Rails::Application) {
{% endhighlight %}

This shows a call to `Class.new` which in Ruby means creating an anonymous class which inherits from the class passed into the new call.

## Background for Class.new

The [Class.new method signature][1] is `Class.new(super_class=Object)` which means that you can specify the superclass by passing in the Class constant or if nothing is specified the default superclass will be Object.

One thing to note with this technique is that the class is unnamed or anonymous until such point that it is assigned to a constant. In the case of The Smallest Rails App the new class is assigned to the constant `TheSmallestRailsApp`.

One use for this would be to create a new class that inherits from another but does not have any body. A common example is when it&#8217;s necessary in a Ruby application to have a domain specific named exception available for rescue. For example:

{% highlight ruby linenos %}
SomeCustomIssue = Class.new(StandardError)
{% endhighlight %}

This way its possible to `rescue SomeCustomIssue` within the application. Using Class.new to define this custom Error class is far cleaner than using the more common class declaration syntax which would look like this:

{% highlight ruby linenos %}
class SomeCustomIssue < StandardError
end
{% endhighlight %}

Class.new can also take a block of code which will can be used to define methods (using `class_eval`) in the new class. For more information on where developers might use Class.new check out [fun with class.new][2] and [when to class.new][3]

## The run command

Working backwards we see the "run" command at the start of the line. This is part of Rack and only works if executed on the command line using the `rackup` command. Usually the Ruby code for a Rack application is saved in a file called `config.ru` (especially in a Rails app) however this is not necessary and can be saved as a ruby file (with .rb extension) too.

Another intesting thing to note is that the run command comes from the [Rack Handler Module][4]. This means that the actual call to run could also be written as this:

{% highlight ruby linenos %}Rack::Handler::Thin.run
{% endhighlight %}

I happen to have the Thin gem installed so this runs fine on my machine. If you don't have Thin installed then you will probably see the error "\`require': cannot load such file -- thin". This is Rack evaluating the code and trying to require thin automatically. Just install the thin gem or use a different Rack::Handler. Of course, just simply call `run` and Rack will use whatever handler it finds on your machine automatically. For details on how Rack does this check out the [Rack source code][5].

One error you might come across when trying to run this Rack application in your environment is "undefined method 'append' for ActionDispatch::Routing::RouteSet". In my case it was becasue the gemset that I was using was using an earlier version of Rails (3.0.x). Switching to another gemset with Rails 3.2.x fixed this issue.

## Refactorings

In the Railscast, Ryan refactors this particular line to the following:

{% highlight ruby linenos %}
class TheSmallestRailsApp < Rails::Application
  #code in the class are on other lines so ommited here
end

run TheSmallestRailsApp
{% endhighlight %}

Note above that I have not included the body code of the class. The above code is essentially the same as line 3 of the Smallest Rails App except expanded to 3 lines. It demonstrates that a call to Class.new creates a class that inherits from the constant passed to it (in this case Rails::Application) and that it is assigned to the constant TheSmallestRailsApp which is then run as a Rack app.

## Conclusion

In line 3 we learned about Class.new, anonymous classes a little about rack applications. Next post will be about the next line of code!

 [1]: http://www.ruby-doc.org/core-1.9.3/Class.html#method-c-new
 [2]: http://blog.rubybestpractices.com/posts/gregory/anonymous_class_hacks.html
 [3]: http://weblog.therealadam.com/2011/12/30/when-to-class-new/
 [4]: http://rack.rubyforge.org/doc/classes/Rack/Handler.html
 [5]: https://github.com/rack/rack

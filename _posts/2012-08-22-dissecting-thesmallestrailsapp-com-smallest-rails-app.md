---
title: Dissecting thesmallestrailsapp.com (Smallest Rails App)
author: jensendarren
layout: post
permalink: /2012/08/dissecting-thesmallestrailsapp-com-smallest-rails-app/
image:
  -
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
tags:
  - advanced
  - programming
  - rails
  - refactoring
  - ruby
  - shortcuts
  - tricks
---
Many of you are probably familiar with the excellent RailsCast on [Rails Modularity][1] where Ryan Bates walks through, what appears to be the 'Smallest Rails App' and then takes a bloated Rails app and slims it down to it's bare bones (but not quite as small as [thesmallestrailsapp.com][2]!).

When I first watched this screencast, I have to admit that I was overwhelmed by the sheer number of Ruby techniques and tricks used to make the applications code so small. I also found it necessary to watch the first part of the screencast several times over in order to get these tricks to sink in. I have to admit that not all are terribly useful in day to day programming, however, I have decided to write a post that breaks some of these concepts down at my own speed. So here we go.

## The smallest require statement

First up is the first line which is the neat require statement:

{% highlight ruby linenos %}
%w(action_controller/railtie coderay markaby).map &method(:require)
{% endhighlight %}

Firstly, in case you don't already know the syntax %w followed by brackets is a way to easily create an array of strings without having to write quotes or commas. So this will create an array like so: ["action_controller/railtie", "coderay", "markaby"]. As you can see, a space in the %w syntax dentoes a new element of the array of Strings.

Now we have that out of the way, what does `map &method(:require)` do? Well in real basic terms, all it does is call the method (`require` in this case) passing in each element of the array to it. If we were to rewrite this line in a more verbose (and perhaps) clearer way it would look like this (more familiar) Ruby code:

{% highlight ruby linenos %}require 'action_controller/railtie'
require 'coderay'
require 'markaby'
{% endhighlight %}

## Another Example

Let me give you another example. Consider the following statement and imagine running it in an IRB session. What would the output be?

{% highlight ruby linenos %}[String, Array, Object].map &method(:is_a?)
{% endhighlight %}

Well, lets break it down. This time we have an array of Ruby constants which we call `map` on and we pass `&method(:is_a?)` as a block which means that each element of the Array will passed as a parameter to the `is_a?` method.

In an IRB session, what will the receiver of the call of `is_a?` be then? A quick call to `self` reveals `main` and `main.class #=> Object`. Add so it should return `[false, false, true]` (since the last element in the array is Object it's the same as calling `main.is_a? Object`. So this single line of code is the same as:

{% highlight ruby linenos %}main.is_a? String #=> false
main.is_a? Array #=> false
main.is_a? Object #=> true
{% endhighlight %}

A little contrived, I know. Another basic example which calls `p "foo"` and `p "bar"` simply outputting the string in the console.

{% highlight ruby linenos %}%w(foo bar).map &method(:p)
{% endhighlight %}

## Uses

So where do you really need to use this (apart from in the smallestrailsapp) I hear you cry!? Well one good example is when you find you are writing the same code as a block over and over, you may think *&#8220;I wish I could extract this into a method and call the method from within the block instead&#8221;*.

So, here's a quick simple example to demonstrate this dream. Say you were finding yourself having to regularly double the elements of an array. You might find this throughout your code:

{% highlight ruby linenos %}#Somewhere in the codesphere:
[1,2,3].map {|e| e * 2}

#Maybe in another line, method Module or Class
[3, 5, 6].map {|e| e * 2}
{% endhighlight %}

What you could do (if you really think you need to) is extract the block into a method and then use the `&method` syntax to call it on each element, like so:

{% highlight ruby linenos %}def double(n)
  n * 2
end

#Now pass double as a block
[1,2,3].map &method(:double) #=> [2,4,6]
{% endhighlight %}

Here is a great post that explains the idea of [passing methods like blocks in Ruby][3].

## Conclusion

This is a great way to write succinct code in Ruby and confuse newbies! I would use it sparingly but definitely worth considering in the above case where you find that you need to reuse a block of code in multiple places.

I really like the smallestrailsapp and I have taken an entire post just to discuss the first line! Can't wait to dig into line 2 (which is empty ;)). Until then, take care and happy refactoring!

 [1]: http://railscasts.com/episodes/349-rails-modularity
 [2]: http://thesmallestrailsapp.com
 [3]: http://alan.dipert.org/post/339368222/passing-methods-like-blocks-in-ruby

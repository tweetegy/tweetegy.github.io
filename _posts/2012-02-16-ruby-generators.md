---
title: Ruby Generators
author: jensendarren
layout: post
permalink: /2012/02/ruby-generators/
image:
  -
seo_follow:
  - 'false'
categories:
  - Ruby Server Programming
tags:
  - closures
  - enumerators
  - fibonacci
  - generators
  - iterators
  - lambda
  - ruby
---
## Simple definition of Generators

(Note there is a full definition available on [Wikipedia][1])

I think the main thing to take away from the Wikipedia definition is that a generator is something* that looks like a function and behaves like an iterator*. What this means is that we can create a generator and call it, however many times we want, and it will return the next value in the iteration. It means (as iterators do) that we can have more control over what happens before and after each iteration.

It also means that developers get the additional power to have complete control over what the iterator actually returns (for example a sequence of Prime Numbers). Another benefit is that they require less memory than say iterating over a large array that&#8217;s already explicitly defined.

## So what does a Generator look like in Ruby?

Since Generators are like Iterators a good example to work with would be to write a program that generates a subset of Prime Numbers or Fibonacci Numbers &#8211; so this is what we will do in Ruby using a number of different approaches.

In Ruby, Enumerators are a form of Generators so here is a sligtly modified example (to use Ruby 1.9.2 Enumerator class instead of the Ruby 1.8.7 Generator) from [Anthony Lewis&#8217;s][2] blog. This example generates a set of Prime Numbers:

{% highlight ruby linenos %}g = Enumerator.new do |g|
  n = 2
  p = []

  while true
    if p.all? { |f| n % f != 0 }
      g.yield n
      p &lt;&lt; n
    end
    n += 1
  end
end

p g.take(10) #=> [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
{% endhighlight %}

Here is another example to generate Fibonacci Numbers that uses the Enumerator class, this is a slightly modified example taken from the [Ruby docs][3]. The only change I made here is that I am calling *yield* instead of using the alias *<<*. You could think of the call to yield or << as like *sending out the value back to the caller* (or at least that&#8217;s the way I think of it when I see a *yield* call in a Generator!)

{% highlight ruby linenos %}fib = Enumerator.new do |y|
  a = b = 1
  loop do
    y.yield a
    a, b = b, a + b
  end
end

p fib.take(10) # => [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
{% endhighlight %}

## Little more advanced example

If you want to build your own Generator, using lambdas, check out this example from [Victor Moroz][4]

{% highlight ruby linenos %}fib =
  lambda{
    a, b = 0, 1
    lambda{ begin a ensure a, b = b, a + b end }
  }[]

puts (0..19).map{ fib[] }.inspect
{% endhighlight %}

## Full definitions of Generators

(Full definition from [Wikipedia][1]): &#8220;A generator is a special routine that can be used to control the iteration behaviour of a loop. A generator is very similar to a function that returns an array, in that a generator has parameters, can be called, and generates a sequence of values. However, instead of building an array containing all the values and returning them all at once, a generator yields the values one at a time, which requires less memory and allows the caller to get started processing the first few values immediately. In short, a generator *looks* like a *function* but behaves like an *iterator*.&#8221;

 [1]: http://en.wikipedia.org/wiki/Generator_(computer_programming)
 [2]: http://anthonylewis.com/2007/11/09/ruby-generators/
 [3]: http://ruby-doc.org/core-1.9.3/Enumerator.html#method-c-new
 [4]: http://blog.vmoroz.com/2011/04/ruby-generators.html

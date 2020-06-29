---
title: Ruby blocks, Procs and lambdas
author: jensendarren
layout: post
permalink: /2012/01/ruby-blocks-procs-and-lambdas/
seo_follow:
  - 'false'
image:
  -
categories:
  - Ruby Server Programming
tags:
  - closures
  - lambdas
  - procs
  - programming
  - ruby
---
## Preamble

One of my favorite parts of the Ruby Programming language is being able to pass code to a method (usually an iterator) and have that code executed dynamically. In Ruby there are several ways to do this using: blocks, Procs and lamdbas. While these are all very similar, there are some very subtle differences. First let me explain blocks.

## Blocks

Blocks are code that can be implicitly passed to a method in Ruby. Any method can in fact be passed a block of code even if the method does not explicitly expect it. In the following example, the method *test_blocks* does not explicitly expect or do anything with a block, however, if called with a block the code will run printing in test_blocks.

{% highlight ruby linenos %}
def test_blocks
  puts "in test_blocks"
end

test_blocks { puts "in the block" }

# Output:
# in test_blocks
{% endhighlight %}

By simply adding the *yield* keyword to the method *test_blocks* the block will then be run causing the output to change:

{% highlight ruby linenos %}
def test_blocks
  puts "in test_blocks"
  yield
end

test_blocks { puts "in the block" }

# Output:
# in test_blocks
# in the block
{% endhighlight %}

## Procs

It is possible to get a handle on the block by explicitly putting the block as a parameter in the method definition. Note that this must be the last parameter and must start with an ampersand:

{% highlight ruby linenos %}
def test_blocks(&block)
  puts block.class
end

test_blocks { puts "in the block" }
# Output:
# Proc
{% endhighlight %}

Notice that the class is *Proc* and so the only difference between a block and a Proc is that a Proc can be passed around as an explicit variable. Here is another example where we create a Proc object first and pass that into another method that expects a Proc:

{% highlight ruby linenos %}
}def test_blocks(some_proc)
  puts some_proc.call
end

some_new_proc = Proc.new { puts "in the Proc !" }
test_blocks(some_new_proc)

# Output:
# in the Proc !
{% endhighlight %}

The main reason for having both *block* and *Proc* is:

*   If we want to write Ruby like syntax when passing code to a method, and we only want to pass a single Proc we use **blocks**. This is very common pattern in Ruby code &#8211; especially with iterators.
*   If we want to save a piece of code in a variable for reuse then use an explicit **Proc** object
*   If we want to pass multiple blocks of code to a method and invoke them, we also must use an explicit **Proc** object

## Lambda

Lambda is very similar to a Proc. Lets first start off with an example:

{% highlight ruby linenos %}
def test_lambda(some_lambda)
  some_lambda.call
end

some_new_lambda = lambda {puts "in the lambda" }
test_lambda (some_new_lambda)

# Output:
# in the Proc !
{% endhighlight %}

As you can see this looks exactly like the Proc example above, so what is the difference? Well the answer is in the way Ruby handles Procs and Lamdbas. There are, in fact, two differences:

1.  Lamdbas check the number of parameters passed into the call, Procs do not check the number of parameters
2.  Lamdbas return from the executing block but **not** from the lexically surrounding method call when they encounter a *return* keyword (in other words lambda behaves like calling a method which then calls return, whereas a Proc will return from the calling method too

Here are a couple of examples to illustrate this. Firstly, the parameter handling differences:

{% highlight ruby linenos %}
def test_parameter_handling(code)
  code.call(1,2)
end

l = lambda {|a,b,c| puts "#{a} is a #{a.class}, #{b} is a #{b.class}, #{c} is a #{c.class}" }
p = Proc.new {|a,b,c| puts "#{a} is a #{a.class}, #{b} is a #{b.class}, #{c} is a #{c.class}" }

test_parameter_handling (p)
test_parameter_handling (l)

# Output:
# 1 is a Fixnum, 2 is a Fixnum,  is a NilClass
# ArgumentError: wrong number of arguments (2 for 3)
{% endhighlight %}

As you can see from the output of the example, the Proc does not care if we only call with two parameters when it's possible to use three parameters it simply sets all unused parameters to *nil*. However, when passing a *lambda* to the method, Ruby throws an *ArgumentError* instead. So if you want your Proc (a lamdba is an instance of Proc, by the way) to be called with a specific set of parameters no more and no less then use *lamdba*.

Here is an example that demonstrates the differences how Proc and lamdba behave when they encounter a *return* statement:

{% highlight ruby linenos %}def return_using_proc
  Proc.new { return "Hi from proc!"}.call
  puts "end of proc call"
end

def return_using_lambda
  lambda {return "Hi from lambda!"}.call
  puts "end of lambda call"
end

puts return_using_proc
puts

# Output
# Hi from proc!
# end of lambda call
{% endhighlight %}

When *return\_using\_proc* is called the method stops processing as soon as it encounters the return statement and we see Hi from proc! output. When *return\_using\_lambda* is called the return is encountered but does not cause the method to return and instead the method continues to run so we see end of lambda call&#8221; output.

Here is another example to demonstrate this in a slightly different way:

{% highlight ruby linenos %}
def test_returns
  puts "top"

  pr = Proc.new { return }
  pr.call

  puts "bottom"
end

puts "before call"
test_returns
puts "after call"

# Output
# before call
# top
# after call
{% endhighlight %}

In this example, we see before call output followed by top as we would expect but then we see after call &#8211; note there is no output for our puts bottom statement. This is because the call to the Proc object (pr.call) caused the calling method to return since the Proc contained a return statement. If we replace the Proc with a lambda then we would see bottom output as well since the call to the lambda simply returns back to the method which called it.

The main reason for this diffence is because Procs are more like blocks of code that can be dropped in and executed whereas lamdbas behave more like methods. It is also for this reason why lambdas are more strict with parameter checking just as methods are.

## Ruby Closures

It's important to note that blocks, Procs and lambdas are all [closures][1]. Basically this means that they hold the values of any variables that were around when they were created. This is a very powerful feature of Ruby because if we wrap creating a block of code in a method call we can dynamically create different behavior. For example:

{% highlight ruby linenos %}
def create_multiplier(m)
  lambda {|val| val * m}
end

two_times = create_multiplier(2)
three_times = create_multiplier(3)

puts two_times.call(1)
puts three_times.call(1)

# Output
# 2
# 3
{% endhighlight %}

## Conclusion

In conclusion we are dealing with blocks of code that can be passed to method calls and executed within that method. When the block of code is executed in the method the state of any variables at the time of creation of that block of code are used. This means that blocks, Procs and lamdbas are all [closures][1] in Ruby.

It's also interesting to conclude that:

**Lambdas** have **strict** parameter checking and **diminutive** returns.
**Procs** have **no** parameter checking and **strong** returns.
**Blocks** and **lambdas** are essentially just anonymous *Procs*

## Some tutorials

*   [Understanding Ruby Blocks, Procs and Lambdas][2]
*   [The Ruby Object Model and Metaprogramming &#8211; Episode 3][3] </ul>

 [1]: http://en.wikipedia.org/wiki/Closure_%28computer_science%29
 [2]: http://www.robertsosinski.com/2008/12/21/understanding-ruby-blocks-procs-and-lambdas/
 [3]: http://pragprog.com/screencasts/v-dtrubyom/the-ruby-object-model-and-metaprogramming

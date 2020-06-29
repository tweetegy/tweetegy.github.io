---
title: Forking Ruby Processes or How to fork Ruby
author: jensendarren
layout: post
permalink: /2012/04/forking-ruby-processes-or-how-to-fork-ruby/
image:
  -
seo_follow:
  - 'false'
categories:
  - Ruby Server Programming
tags:
  - fork
  - multitask
  - processes
  - ruby
  - threading
  - threads
---
Forking is a UNIX term that makes a process copy itself programmatically. The definition, on good old [Wikipedia][1] is: *A fork in a multithreading environment means that a thread of execution is duplicated, creating a child thread from the parent thread.*. Note this does not necessarily mean that a copy of the memory is allocated. In many cases the memory is shared until one of the processes actually makes a change to something then the memory: this is called copy-on-write (CoW).

So with the UNIX terminology out of the way, how does this work in Ruby? How can we fork a Ruby process using Ruby? Here is a basic example:

{% highlight ruby linenos %}puts "This is the first line before the fork (pid #{Process.pid})"
puts fork
puts "This is the second line after the fork (pid #{Process.pid})"
{% endhighlight %}

The output from this code might be something like:

> This is the first line before the fork (pid 10207)
> 10209
> This is the second line after the fork (pid 10207)
> This is the second line after the fork (pid 10209)

So what just happened? Well the first line is pretty clear but then what is the output on the second line (10209)? Well that is the PID returned by the call to fork and , of course, represents the PID of the child process. Then we see the next two lines; one executed by the parent (with PID = 10207) and the other executed by the child (with PID = 10209).

## Blocks

Very often, the code that developers want to run in a separate process is passed as a block to the fork method, like so:

{% highlight ruby linenos %}puts "You can also put forked code in a block pid: #{Process.pid}"
fork do
  puts "Hello from fork pid: #{Process.pid}"
end
puts "The parent process just skips over it: #{Process.pid}"
{% endhighlight %}

This outputs the three lines (sometimes) in different order but usally like this:

> You can also put forked code in a block pid: 10161
> The parent process just skips over it: 10161
> Hello from fork pid: 10163

Note that the PID in the block is different from the PID outside it. This is because the code running within the block is running from within the new process created by using fork.

## Multi-core CPU test

Let&#8217;s quickly verify one of the main benefits for using fork which is for multi-processing. Let&#8217;s write a test Ruby program that can be used to demonstrate this effect.

{% highlight ruby linenos %}def cpu_intensive_process
  puts "Pid: #{Process.pid}"
  x = 0
  10000000.times do |i|
    x = i + x
  end
end

fork
cpu_intensive_process #should see two processors flat out!
{% endhighlight %}

When this code is run two processors should max out. If the machine has more than two processors available then **only two processors will ever max out** with this code as it stands. However, using fork additional times will result in even more processes going flat out (up to all your processors, of course) ! Here is a screen capture of the CPU process monitor on my PC:
![CPU Maxing out during a Ruby Fork](/assets/cpu-maxing-out.png)

## Memory test

So what happens with the memory allocation. Apparently in Ruby 1.9.3 (which I am using here) there is no CoW functionality which means that the memory allocation for each process should be the same. Here is a little program that goes some way towards testing that (although, I would say that it is not conclusive):

{% highlight ruby linenos %}
hash = Hash.new #load up the memory a little

1000000.times do |i|
  hash[i] = "foo"
end

puts "Hash contains #{hash.keys.count} keys"

def show_memory_usage(whoami)
  pid = Process.pid

  mem = `pmap #{pid}`

  puts "Memory usage for #{whoami} pid: #{pid} is: #{mem.lines.to_a.last}"

  sleep #keep the process alive
end

puts "Now lets fork this process and see what memory is allocated to the child"

puts "Before..."

if fork
  show_memory_usage("parent")
else

  puts "After..."

  1000000.times do |i| #change the values in the child memory allocation
    hash[i] = "bar"
  end

  show_memory_usage("child")
end
{% endhighlight %}

Here is the result of this test. Feel free to copy this code, changing it around a bit and see what results you get!

> Hash contains 1000000 keys
> Now lets fork this process and see what memory is allocated to the child
> Before&#8230;
> After&#8230;
> Memory usage for parent pid: 10291 is: total 62592K
> Memory usage for child pid: 10293 is: total 73164K

## Orphan Processes

Finally, lets try out testing orphan processes and how they behave. Run this final little program to see:

{% highlight ruby linenos %}fork do
  5.times do
    sleep 1
    puts "I'm an orphan!"
  end
end
abort "Parent process died..."
{% endhighlight %}

The results are as follows:

> Parent process died;
> terminal ~: I'm an orphan!
> I'm an orphan!
> I'm an orphan!
> I'm an orphan!
> I'm an orphan!

So what's happening here? Well the parent process completes and therefore terminates to give me back the terminal prompt. But then suddenly I am interupted with the output of the 5 child processes (now technically orphans) as they execute their code one after the other following a little sleep!

Well that's all for my post today on Ruby Forks or how to Fork Ruby. Next time we will be having more fun with Ruby (but not necessary forking it). Till then, happy *(insert whatever it is that you do here)*!

 [1]: http://en.wikipedia.org/wiki/Fork_%28operating_system%29

---
title: (Almost) Everthing an experienced programmer needs to know to get started with PHP
author: jensendarren
layout: post
permalink: /2011/01/almost-everthing-an-experienced-programmer-needs-to-know-to-get-started-with-php/
seo_follow:
  - 'false'
image:
  -
categories:
  - PHP
  - Programming
tags:
  - experienced
  - new
  - php
  - programming
  - tips
---
I consider myself to be an experienced programmer. Having spent the last 10 years working on a variety of platforms like Windows and Linux, using different programming languages like Java, C#, VB, Ruby, JavaScript etc, I feel that I have a broad experience and understanding of programming. However, there is always that moment when one must use a different language, for whatever reason. In my case I had to start tinkering with a site developed by someone else in PHP. Now I have always tried to avoid PHP since I was very much into compiled languages like Java and C#. Since breaking away from the corporate world, I have been able to move to scripting languages like Ruby, JavaScript and now PHP.

What I want to do is briefly summarize what an experienced programmer needs to know to get started with PHP. I managed to build a simple site in PHP in less than one week using PHP / MySQL / Apache on Linux (commonly known as LAMP stack). I will just list some of the things that I found useful during the journey. The site I built by the way is for my very own [Computer School in Cambodia][1] (shameless personal project plug, I suppose, but only if your in Cambodia .

Note: To place PHP script inside a page you need to open with <? and close with ?>

**Setting variables:**

{% highlight php %}
$somevar = "hello world";
{% endhighlight %}

**String concatenation is done with dots!**

{% highlight php %}
$url="/somthing/".$id."/".$name.".html";
{% endhighlight %}

**Connecting to a database**

{% highlight php %}
mysql_connect(localhost,$username,$password);
mysql_select_db($database) or die( "Unable to select database");
{% endhighlight %}

**Execute a query and count results**

{% highlight php %}
$results=mysql_query($query_sql_string);
$num_results=mysql_numrows($results);
{% endhighlight %}

**Get a value from the resultset**

{% highlight php %}
//Note this can be in a while loop.
//$i is a counter.
$id=mysql_result($results,$i,"id");
{% endhighlight %}

**IF statements**

{% highlight php %}
if (!is_null($id)) { // do something }
{% endhighlight %}

**Output values in HTML rendering**

{% highlight php %}
<h1 align="center"><?=$some_value?></h1>
{% endhighlight %}

**Date formatting**

More details of [date formatting in PHP][2]

{% highlight php %}
$date_obj->format("D j M Y")
{% endhighlight %}

**Server Side Includes**

{% highlight php %}
include('db.php');
{% endhighlight %}

 [1]: http://www.cambodiahorizon.com
 [2]: http://adminschoice.com/php-date-format

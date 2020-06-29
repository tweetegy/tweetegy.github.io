---
title: Getting started with the HTML5 Canvas API
author: jensendarren
layout: post
permalink: /2011/10/getting-started-with-the-html5-canvas-api/
image:
  -
seo_follow:
  - 'false'
categories:
  - HTML5 Programming
tags:
  - canvas
  - html5
---
The Canvas API essentially allows the developer to draw on a web page using a JavaScript API. Now you might think, what is the point of this? If you want to show some artwork or animation why not create this first and then send it to the browser as a file?

## The main benefits of using the Canvas API

*   The JavaScript to render the image / animation may be smaller than downloading a file
*   The Canvas API allows for some cool extras like scaling, rotation and even pixel by pixel image access
*   Being an API means that it is programmable, so you can build applications and games purely based on the Canvas API
*   The Canvas API is potentially powerful enough that you could [write an image editing suite that runs in the browser][1] (no more Photoshop required!)
*   The Canvas API does not require any browser plug-ins (bye, bye Falsh!)

## The Basics

Eveytime you use the canvas API there are always some things you must do. The first thing is to use the DOM to get a reference to the Canvas object:

{% highlight javascript %}
var canvas = document.getElementById("my-id");
{% endhighlight %}

Then we must get the 2D context of the Canvas object like so: (in the near future a 3D context will become more widely available)

{% highlight javascript %}var context = canvas.getContext("2d");
{% endhighlight %}

At this point we have access to the 2D context of our canvas object, so we can begin to call methods on this. The following example draws a line on the canvas:

{% highlight javascript %}context.beginPath();
context.moveTo(10,10);
context.lineTo(150,150);
//Need to call stroke() function otherwise nothing will appear!
context.stroke();
{% endhighlight %}

## Drawing shapes

You can do much more with the Canvas API than drawing lines. You can:

*   Draw and fill shapes
*   Draw lines
*   Draw curves
*   Add images
*   Transform objects
*   &#8230;and more!

Here is an example of drawing circle using the arc function:

{% highlight javascript %}context.arc(0,0,180,7,0.71);
context.lineWidth = 5;
context.stroke();
{% endhighlight %}

Here is a full example (JavaScript within the HTML Script tag) that uses the quadraticCurveTo function to draw a colorful arc! This example shows the HTML5 Canvas element as well as the full method to get the object and the context and draw the arcs. Notice that it is possible to set many properties on the context object like lineWidth. strokeStyle etc

{% highlight html %}
<canvas id="rainbow" style="border: 1px solid;" width="470" height="400" />
{% endhighlight %}

When run in a compatible browser the output should look like so:
![Arc Canvas API Example](/assets/arc-canvas-example.png)

That's it for this very basic introduction! If you want to see the full power of the HTML5 Canvas API then check out some of the (really) cool examples below!

## Some cool examples

*   [Asteroids Game][3]
*   [Starfield][4]
*   [Landscapes][5]
*   [Cloth Experiment][6]
*   [Magnetic Experiment][7]

## Some books and tutorials

*   [html5canvastutorials.com][8]
*   [Canvas Deep Dive][9]
*   [HTML5 Canvas book from Oreilly][10]

 [1]: http://code.google.com/p/paintweb/
 [2]: http://www.tweetegy.com/wp-content/uploads/2011/10/Screenshot.png
 [3]: http://www.kevs3d.co.uk/dev/asteroids/
 [4]: http://arapehlivanian.com/wp-content/uploads/2007/02/canvas.html
 [5]: http://www.effectgames.com/demos/canvascycle/
 [6]: http://andrew-hoyer.com/experiments/cloth/
 [7]: http://hakim.se/experiments/html5/magnetic/02/
 [8]: http://www.html5canvastutorials.com/
 [9]: http://projects.joshy.org/presentations/HTML/CanvasDeepDive/presentation.html
 [10]: http://shop.oreilly.com/product/0636920013327.do

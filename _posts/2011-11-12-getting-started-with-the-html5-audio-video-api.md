---
title: Getting started with the HTML5 Audio Video API
author: jensendarren
layout: post
permalink: /2011/11/getting-started-with-the-html5-audio-video-api/
image:
  -
seo_follow:
  - 'false'
categories:
  - HTML5 Programming
tags:
  - api
  - audio
  - html5
  - video
---
The HTML5 Audio and Video API is a real game changer. For starters, think about how this affects the (once) mighty Flash. For years Flash was the most popular way to distribute audio and especially video via the web to a browser. Now that can change with HTML5. Even Adobe has announced that it is [to stop developing Flash for mobile][1]

## The Basics

Getting started with the Audio and Video API is real easy (as is everything in HTML5, in fact!). At the very basic level all you need to do is declare an <audio> or <video> tag in your html like so:

{% highlight html %}
  <video src="media/some-video-file.mp4"></video>
{% endhighlight %}

This is so basic that it does not even render the controls. In order to control video playback, the user has to use the context menu (which is not so easy to find for some). In order to render controls simply add the controls attribute like so:

{% highlight html %}
  <video src="media/some-video-file.mp4" controls></video>
{% endhighlight %}

Now we see the controls appear as sown below (notice that the first frame of the video is shown by default):

![HTML5 Video with controls](/assets/html5-video-with-controls-300x226.jpg)

Simple! So why does HTML5 have the option to not show the controls? Well the reason is to give the developer control over how the controls are rendered. Since JavaScript has an API to access the Audio and Video instances within the browser it is possible to create and style a &#8220;Play/Pause&#8221; button in anyway you like! Here is some example JavaScript code that demonstrates how to do this:

{% highlight javascript linenos %}
function playPauseMusic() {
  var music = document.getElementById("music");
  var playPauseBtn = document.getElementById("playPauseBtn");
  if (music.paused) {
    music.play();
    playPauseBtn.innerHTML = "Pause the Music!";
  }
  else {
    music.pause();
    playPauseBtn.innerHTML ="Play the Music!";
  }
}
{% endhighlight %}

As you can see this is quite easy to do. Simply add an audio tag to the html page with id=&#8221;music&#8221; and a button tag with id=&#8221;playPauseBtn&#8221; and this JavaScript can be used to control the play / pause of your music! All it does is render a button but it works and demonstrates how easy it is to use the (in this case) Audio API (the Video API is very similar, by the way).

## Use the Source!

So far I have only shown some simple examples. It is possible in HTML5 to specify different sources so that you can provide different file types for the media you want rendered. So in order to increase the chance that the browser supports the particular container or codec, make sure you save your media as different file types (e.g. ogg, mp4 etc) and use the source as in the following audio example:

{% highlight html %}
<audio>
  <source src="1.ogg" >
  <source src="1.mp3" >
  Some music by someone!
</audio>
{% endhighlight %}

Note the text &#8216;Some music by someone!&#8217; just before the closing <audio> tag. This text will be rendered in non-supporting browsers so it provides a nice and easy way to gracefully fallback if that is the case.

## Autoplay attribute

HTML5 Audio and Video API comes with the optional autoplay attribute. Use this with caution! There is nothing more annoying than a website that starts playing audio or video automatically on load! However, if you must, simply include the attribute in the <audio> or <video> tag.

## Video API Basic Tip: Play Video on Mouseover

One thing you might have seen on the web is the concept of playing a video in a thumbnail when the mouse hovers over the video image. This is surprisingly easy to do in HTML5, by the way! All you need to do is set the onmouseover and onmouseout events to call play() and pause() methods respectively, like so:

{% highlight html %}
<video id="movies" onmouseover="this.play()" onmouseout="this.pause()" autobuffer="true">
<!-- video source here -->
</video>
{% endhighlight %}


That&#8217;s it for this very basic introduction! If you want to learn more about the HTML5 Audio and Video API, check out some of the following resources:

## Some cool examples

*   [Basic Examples][2]
*   [Video Events Example][3]
*   [Some notes on HTML5 Video Codes support][4]

## Some tutorials

*   [A great HTML5 Audio Video API tutorial by Mozilla][5]
*   [Another great tutorial, this time from HTML5 Rocks!][6]

 [1]: http://searchenginewatch.com/article/2124277/Adobe-To-Stop-Developing-Flash-For-Mobile
 [2]: http://shapeshed.com/examples/HTML5-video-element/
 [3]: http://double.co.nz/video_test/events.html
 [4]: http://www.808.dk/?code-html-5-video
 [5]: https://developer.mozilla.org/en/Using_audio_and_video_in_Firefox
 [6]: http://www.html5rocks.com/en/tutorials/video/basics/

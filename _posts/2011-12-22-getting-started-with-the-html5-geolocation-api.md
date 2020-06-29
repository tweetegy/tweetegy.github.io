---
title: Getting started with the HTML5 Geolocation API
author: jensendarren
layout: post
permalink: /2011/12/getting-started-with-the-html5-geolocation-api/
image:
  -
seo_follow:
  - 'false'
categories:
  - HTML5 Programming
tags:
  - api
  - geolocation
  - html5
---
Geolocation presents major opportunities for mobile application development. It's an exciting way to work with location based applications and services and really give the user something that is of great value to them. By using Geolocation, developers can find out the location of the user, can give the user directions to other places, can tell the user where the nearest XYZ place might be, can tell the user how far they have moved in a given timeframe and more, and more.

## The Basics

When using the Geolocation API it is important to know the data that we are working with. With Geolocation we work with Latitude and Longitude values. These represent the following:

*   **Latitude**: The distance North or South of the equator
*   **Longitude**: The distance East or West of Greenwich, England

Usually, these values come from the mobile device GPS or, if the GPS is not available then the co-ordinates can be inferred from the IP address, Wi-Fi or even entered directly by the user.

## Privacy

Since we are dealing with sensitive information, there are strict privacy requirements that must be followed when working with the Geolocation API. Fortunately, privacy user confirmation is triggered in all browses (that support Geolocation) automatically on any API call. You may have seen the message in some applications asking if it is ok that the application uses your location. This is Geolocation API security kicking in! It happens only once per domain so once you accept you will not need to accept every time you perform a Geolocation based action on that site.

## Getting the co-ordinate data

To get the current position of the user, simply execute this function. Note that *updateLocation* is a callback function to handle the result.

{% highlight javascript %}
navigator.geolocation.getCurrentPosition(updateLocation)
{% endhighlight %}

The position object passed to the *updateLocation()* function contains the following data attributes:

*   latitude
*   longitude
*   accuracy

Here is an example of how the updateLocation function might appear:

{% highlight javascript %}
function updateLocation(position) {
    var latitude = position.coords.latitude;
    var longitude = position.coords.longitude;
    var accuracy = position.coords.accuracy;
    var timestamp = position.timestamp;
    document.getElementById("latitude").innerHTML = latitude;
    document.getElementById("longitude").innerHTML = longitude;
    document.getElementById("accuracy").innerHTML = accuracy;
    document.getElementById("timestamp").innerHTML = timestamp;
}
{% endhighlight %}

## Handling Errors

It's important to handle errors in Geolocation API calls because there is so much potential for things to go wrong: network error, GPS issues, privacy / security issues, timeout etc. We can handle errors using another callback function that takes an error code and deals with that code accordingly. What we need to do first is include that callback function in the call to the *getCurrentPosition* Geolocation function:

{% highlight javascript %}navigator.geolocation.getCurrentPosition(updateLocation, handleError)
{% endhighlight %}

Then our *handleError* callback function might look like this (notice the 4 possible error codes):

{% highlight javascript %}
function handleError(error) {
  switch (error.code) {
    case 0:
	    updateStatus("There was an error while retrieving your location: " + error.message);
	    break;
    case 1:
	    updateStatus("The user prevented this page from retrieving a location.");
	    break;
    case 2:
	    updateStatus("The browser was unable to determine your location: " + error.message);
	    break;
    case 3:
	    updateStatus("The browser timed out before retrieving the location.");
 	    break;
  }
}
{% endhighlight %}

## Repeated requests to keep track of users position

In the example above, *getCurrentPosition() *makes one call to get the users location. However, there maybe situations where you will want to track the users *change* in position over time. In this case it is better to use *watchPosition()*.

Fortunately, it's really simple to change to use watchPosition, here is an example so you can see how similar it is. Note we can set the *watchId* variable so that we can tell the API to stop watching by making a call to *clearWatch* function at a later time:

{% highlight javascript %}
var watchId = navigator.geolocation.watchPosition(updateLocation, handleError);
//Do something and sometime later we can stop watching by passing the watchId to clearWatch():
navigator.geolocation.clearWatch(watchId);
{% endhighlight %}

## Conclusion

While this was a very basic introduction to the HTML5 Geolocation API, it is enough for developers to get started building something useful! Remember that you can also use Google Maps API to integrate mapping data into your application too!

## Some cool examples

*   [Basic Example][1]
*   [Demo&#8217;s difference with IP and Geolocation API based calls][2]

## Some tutorials

*   [Basic Tutorial][3]
*   [Introduction Tutorial][4]
*   [More detailed tutorial][5]

 [1]: http://merged.ca/iphone/html5-geolocation
 [2]: http://www.ip2location.com/html5geolocationapi.aspx
 [3]: http://mobile.tutsplus.com/tutorials/mobile-web-apps/html5-geolocation/
 [4]: http://developer.practicalecommerce.com/articles/2066-An-Introduction-to-HTML5-Geolocation
 [5]: http://www.netmagazine.com/tutorials/link-users-geolocation-data-html5

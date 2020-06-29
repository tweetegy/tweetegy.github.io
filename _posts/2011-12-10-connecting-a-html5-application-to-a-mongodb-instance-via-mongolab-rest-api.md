---
title: Connecting a HTML5 application to a MongoDB instance via MongoLab REST API
author: jensendarren
layout: post
permalink: /2011/12/connecting-a-html5-application-to-a-mongodb-instance-via-mongolab-rest-api/
image:
  -
seo_follow:
  - 'false'
categories:
  - HTML5 Programming
  - JavaScript Client Programming
  - nosql
  - REST Programming
tags:
  - api
  - html5
  - mongodb
  - mongolabs
  - rest
---
## Overview

I needed a free, document based, online data store so that I could quickly build a HTML5 prototype. As an exercise, I quickly whipped up a simple application that can store basic contact details of people.

## Getting Started with MongoLab

To get started with MongoLab is really easy. Simply create your account, your database and your first collection and your good to go. The only reference document you need for this exercise is the [MongoLab REST API docs][1]

## First things first: Adding data to your MongoDB using REST

I am only going to show you the absolute basics here, so no security, validation, callbacks or fancy UI designs &#8211; just the basics! I built this by creating a simple HTML5 form and I used XMLHttpRequest API to send the data to MongoLab.

Here are the code snippets, first the HTML:

{% highlight html %}
<form id="myform" name="myform">
  <input id="name" type="text" name="name" placeholder="Name"><br>
  <input id="email" type="email" name="email" placeholder="Email"><br>
  <input type="button" onclick="send(this.form);" value="Save">
</form>
{% endhighlight %}

&#8230;and the JavaScript. Note the variable *json* which is not defined in this snippet but is basically a JSON representation of the form data. The easiest way to get your form data into JSON is to use JQuery (again, I am not including how to do that to keep us focused on how to interact with the MongoLab RESTful API). Note: you will need to replace YOUR-DATABASE, YOUR-COLLECTION and YOUR-API-KEY with your values.

{% highlight javascript %}var xhr = new XMLHttpRequest();
    xhr.open("POST", "https://api.mongolab.com/api/1/databases/YOUR-DATABASE/collections/YOUR-COLLECTION?apiKey=YOUR-API-KEY", true);
    xhr.setRequestHeader("Content-Type", "application/json");
    xhr.send(json);
{% endhighlight %}

When you load your page you will be able to add a new record to the database. You can check that it has arrived at the database by logging into mongolab.com and viewing the collection.

## Next up: Display a collection of items

It&#8217;s really easy to get data from the MongoLab API &#8211; a simple GET request will do, in fact! Apart from that, this step is really an exercise in using the JavaScript DOM (which I will not cover here). Here is the HTML &#8211; as you can see it is just a placeholder for the data. I am using UL but, of course, you can load the data into whatever placeholder you like.

{% highlight html %}
<ul id="result">
</ul>
{% endhighlight %}

Basic JavaScript snippet shown below. As you see just a simple GET request (and send null since there is no payload). To complete this you&#8217;ll want to add an *onreadystatechange* function to detect when the request.status is 200 and then call your DOM rendering function at that point.

{% highlight javascript %}var request = new XMLHttpRequest();
    request.open("GET", "https://api.mongolab.com/api/1/databases/YOUR-DATABASE/collections/YOUR-COLLECTION?apiKey=YOUR-API-KEY");
    request.send(null);
{% endhighlight %}

## View a single document

To render a single document in your HTML5 application, you&#8217;ll want to, again, use the GET request but this time passing in the document id in the query string and using that to pull the correct document from your MongoDB. The HTML snippet is the same as for the collection, so I&#8217;ll just show the JavaScript code below. Notice how to get the id from the query string. This exampled is hard coded, and thus assumes the id is first in the query string and is in the form &#8220;id=12345&#8243;.

{% highlight javascript %}var queryString = window.top.location.search.substring(1);
    id = queryString.substring(3,queryString.length);
    var request = new XMLHttpRequest();
    request.open("GET", "https://api.mongolab.com/api/1/databases/YOUR-DATABASE/collections/YOUR-COLLECTION/" + id + "?apiKey=YOUR-API-KEY");
    request.send(null);
{% endhighlight %}

## Edit a document

Since this is a RESTful API, to update a document we need to use a PUT verb. The HTML snippet is almost the same as the adding data example, so just the JavaScript snippet here. Notice the getId() function call which simply gets the id from the query string. Obviously this is not a production ready or secure way of getting the id for a document we want to update.

{% highlight javascript %}var xhr = new XMLHttpRequest();
    xhr.open("PUT", "https://api.mongolab.com/api/1/databases/YOUR-DATABASE/collections/YOUR-COLLECTION/" + getId() + "?apiKey=YOUR-API-KEY", true);
    xhr.setRequestHeader("Content-Type", "application/json");
    xhr.send(json);
{% endhighlight %}

## Delete a document

Finally, we want to be able to delete a document. This is really easy as shown. Notice you need to send *null* like before because there is no payload.

{% highlight javascript %}var request = new XMLHttpRequest();
    request.open("DELETE", "https://api.mongolab.com/api/1/databases/YOUR-DATABASE/collections/YOUR-COLLECTION/" + id + "?apiKey=YOUR-API-KEY");
    request.send(null);
{% endhighlight %}

## Conclusion

Thankfully, with the excellent service provided by MongoLab.com developers are able to build nosql document database driven applications in no time and at no cost!

 [1]: http://support.mongolab.com/entries/20433053-rest-api-for-mongodb

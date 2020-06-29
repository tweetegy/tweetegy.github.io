---
title: How to get a semantic list of all countries and locations in the world
author: jensendarren
layout: post
permalink: /2011/02/how-to-get-a-semantic-list-of-all-countries-and-locations-in-the-world/
image:
  -
seo_follow:
  - 'false'
categories:
  - Semantic Application Programming
tags:
  - api
  - countries
  - geonames
  - global
  - locations
  - rest
  - semantic
  - web
  - world
---
I am in the process of building a semantic web application. Since the application I am building will be used globally, I wanted to start with including a semantic collection of all countries in the world. It turns out to be rather simple thanks to the GeoNames API. I am a big fan of JSON so all the examples I give here will return the results as JSON.

## Complete list of all Countries

Get a complete list of all countries is as simple as making the following request (note in all the examples the username is *demo*. In a real application you would need to create your own account with GeoNames):

{% highlight bash %}
http://api.geonames.org/countryInfoJSON?username=demo
{% endhighlight %}

## Get data for a single Country

If you want data for a **single country** then you must include the 2 letter [ISO_639][1] code in your request. In the following example, the request returns data for Cambodia (iso_639 code KH):

{% highlight bash %}
http://api.geonames.org/countryInfoJSON?country=KH&#038;username=demo
{% endhighlight %}

By the way, you will notice that the GeoNames API contains a [rich set of data][2].

## Locations within a Country

GeoNames API has the ability to drill down into various administrative levels (aka *populated places*) within the country via the [GeoNames Children API][3]. To get all the children of a particular country you need to first get the geonameId from [GeoNames Country Info API][4] (discussed above already), and pass that as a parameter to the Children API. The following example gets the first level of children for Cambodia (which happens to have a geonameId of 1831722)

{% highlight bash %}http://api.geonames.org/childrenJSON?geonameId=1831722&#038;username=demo

{% endhighlight %}

To drill down into the next level of children you recursively pass in the geonameId of each child until the API returns an empty result. If you try the GeoNames API with UK data then the first level of children are England, Scotland, Wales etc then if you pass the geonameId of England you will get all the counties of England like Sussex, Kent, Essex, Nottinghamshire etc. Continue to pass in the geonameId until there are no more children and you will have all the locations in the UK (at least according to GeoNames).

I hope you find this useful. In my next post I will discuss how to get language data from [dbpedia][5] using their RESTful API or a custom SPARQL query.

 [1]: http://en.wikipedia.org/wiki/ISO_639
 [2]: http://www.geonames.org/export/codes.html
 [3]: http://www.geonames.org/export/place-hierarchy.html#children
 [4]: http://www.geonames.org/export/web-services.html#countryInfo
 [5]: http://dbpedia.org

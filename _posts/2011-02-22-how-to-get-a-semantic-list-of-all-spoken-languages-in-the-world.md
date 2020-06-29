---
title: How to get a semantic list of all spoken languages in the world
author: jensendarren
layout: post
permalink: /2011/02/how-to-get-a-semantic-list-of-all-spoken-languages-in-the-world/
image:
  -
seo_follow:
  - 'false'
categories:
  - Semantic Application Programming
tags:
  - api
  - dbpedia
  - geonames
  - language
  - languages
  - linkeddata
  - semantic
  - semantics
  - sparql
  - web
---
First stop, grab and parse the list from here: [http://en.wikipedia.org/wiki/List\_of\_ISO\_639-1\_codes][1].

**Getting language meta data by url dereferencing**

This is the easiest method but has less control over the data that is returned (by default all data about the language is returned). Simply replace the [language_name] part of the following url with the value in &#8220;Language Name&#8221; (NOTE: it is case sensitive so the Language Name must be in capital case):

{% highlight bash %}
http://dbpedia.org/resource/[Language_Name]_language.json
{% endhighlight %}

So for Aramaic the url would be:

{% highlight bash %}
http://dbpedia.org/data/Aramaic_language.json
{% endhighlight %}

For Khmer the url would be:

{% highlight bash %}
http://dbpedia.org/data/Khmer_language.json
{% endhighlight %}

**Getting a subset of data**

Let&#8217;s say we want a little more control and therefore need only a subset of the data available. In this case we can turn to the SPARQL endpoint available at [http://dbpedia.org/sparql][2]. Copy and paste the following example to get name, abstract, depiction, comment and label data for English Language:

{% highlight sql %}
PREFIX dbprop: &lt;http://dbpedia.org/property/&gt;
PREFIX dbowl: &lt;http://dbpedia.org/ontology/&gt;
PREFIX dbpedia: &lt;http://dbpedia.org/ontology/&gt;
PREFIX rdfs: &lt;http://www.w3.org/2000/01/rdf-schema#&gt;
PREFIX foaf: &lt;http://xmlns.com/foaf/0.1/&gt;
SELECT ?uri ?name ?label ?thumbnail ?depiction ?abstract ?comment
WHERE {
	?uri dbprop:name "English"@en;
	rdf:type dbpedia-owl:Language;
	dbprop:name ?name;
	foaf:depiction ?depiction;
	dbowl:thumbnail ?thumbnail;
	dbowl:abstract ?abstract;
	rdfs:comment ?comment;
	rdfs:label ?label .
	filter (lang(?label) = "en" )
	filter (lang(?abstract) = "en" )
	filter (lang(?comment) = "en" )
}
{% endhighlight %}

Simply change `?uri dbprop:name "English"@en;` to the (English) name of the language that you want to search for e.g. `?uri dbprop:name "French"@en;` will return the dbpedia data fields related to the French Language.

**Resources**

*   SPARQL: <http://sparql.org/>
*   Linked Data: <http://linkeddata.org/>

 [1]: http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
 [2]: http://dbpedia.org/sparql/

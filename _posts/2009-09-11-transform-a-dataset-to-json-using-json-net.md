---
title: Transform a DataSet to JSON using Json.NET
author: jensendarren
layout: post
permalink: /2009/09/transform-a-dataset-to-json-using-json-net/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
tags:
  - .net
  - convert
  - DataSet
  - DataTable
  - JSON
  - JSON.NET
---
It was my duty recently to convert some data from SQL Server 2005 into JSON. Initially, I started writing a bunch of UDF&#8217;s in SQL Server that built up a string representation of the data in JSON format. The problem with this method is that it is fiddly, slow and does not take into account JSON data types. So I turned to [ Json.NET by James Newton-King][1]. What I liked about this library is it&#8217;s support for LINQ and XML.

Here is a basic method for converting a DataSet that contains one table into JSON. Note the call to AsEnumerable() on the DataTable so that we can then use the LINQ support provided by Json.NET.

{% highlight csharp %}
private static string ConvertDataToJson()
{
    JObject trans =
      new JObject(
        new JProperty("data",
          new JArray(
            from r in ds.Tables[0].AsEnumerable()
            select new JObject(
                ConvertRowToJPropertyArray(r)
              ))));
    return trans.ToString();
}
{% endhighlight %}

And the ConvertRowToJPropertyArray method looks like this:

{% highlight csharp %}private static JProperty[] ConvertRowToJPropertyArray(DataRow dr)
{
	JProperty[] o = new JProperty[dr.Table.Columns.Count];
	for (int i=0; i&lt;dr.Table.Columns.Count; i++) {
		o[i] = new JProperty(dr.Table.Columns[i].ColumnName, dr[dr.Table.Columns[i]]);
}
return o;
{% endhighlight %}

In the [next post][2] I will consider a DataSet that contains multiple tables and relations.

 [1]: http://james.newtonking.com/pages/json-net.aspx
 [2]: http://www.www.tweetegy.com/2009/10/transform-a-dataset-to-json-using-json-net-part-2/

---
title: Transform a DataSet to JSON using Json.NET (Part 2)
author: jensendarren
layout: post
permalink: /2009/10/transform-a-dataset-to-json-using-json-net-part-2/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
tags:
  - .net
  - DataSet
  - DataTable
  - JSON
  - JSON.NET
  - Transformations
---
Here is how to transform a DataSet which contains multiple DataTable and DataRelations, well at least, one way you can do it! See [Transform a DataSet to JSON using Json.NET (Part 1)][1] for details on how to transform a single DataTable into JSON using Json.NET. ****

**Note that this is designed to work when loading one root row at a time. For example, if you have a database of people and there were related tables for work experience and work experience details then you would load one person and all their related work experience into the DataSet and then use this solution to convert it into JSON.**

The DataSet might be designed as follows:

![How a basic person with work experience might look as a DataSet. Note: You do not need to specify any attributes except the primary and foreign keys!](/assets/person-dataset.jpg)

We start with the ConvertDataToJson() method which has the same principal purpose as the example in Part 1 except it calls another method GetRelatedData() to return the JSON. Therefore the ConvertDataToJson() is straightforward and looks as follows:

{% highlight csharp %}private string ConvertDataToJson()
{
JObject data =
	new JObject(
		GetRelatedData()
	);
return data.ToString();
}
{% endhighlight %}

The GetRelatedData() method converts the related data into JSON. We first need to calculate the number of related DataTables in the DataSet. That gives us the size we need for our JProperty array. Then, as we loop through each DataTable, we set the name of each JProperty to the name of the DataTable. Note, however, that we check if ds.Tables[i].ParentRelations.Count is greater than 0 before creating a new JArray. This is because we only want to convert the related data into a JArray once and therefore have it be part of a related JSON element once.

If the current table does not have any parent relations we create a JArray and using Linq we enumerate through each row in the table calling ConvertRowToJObject(r) and GetChildRows(r). Here is the complete method:

{% highlight csharp %}
private JProperty[] GetRelatedData()
{
  //relatedDataArraySize can be passed into this class
  //it is calculated using the following formula
  //relatedDataArraySize = Total number of DataTable - Number of DataTable with DataRelations
  JProperty[] o = new JProperty[relatedDataArraySize];
  int subtractor = 0;
  for (int i = 0; i < ds.Tables.Count; i++)
  {
      if (ds.Tables[i].ParentRelations.Count > 0)
      {
          subtractor += 1;
      }
      if (ds.Tables[i].ParentRelations.Count == 0)
      {
        //Create a JPropery -> JArray
        o[i - subtractor] = new JProperty(ds.Tables[i].TableName,
          new JArray(
            from r in ds.Tables[i].AsEnumerable()
            select new JObject(
              //See Part 1 for an explaination of the ConvertRowToJObject method
              ConvertRowToJObject(r),
              //See below for details of the GetChildRows method
              GetChildRows(r)
                )));
      }

  }
  return o;
}
{% endhighlight %}

In case your interested the code to calculate the relatedDataArraySize is:

{% highlight csharp %}foreach (DataTable t in ds.Tables)
{
	if (t.ChildRelations.Count &gt; 0) numberOfTablesWithRelations += 1;
}
{% endhighlight %}

The GetChildRows(DataRow) method is as follows. Notice that it is a recursive method so that it will keep calling itself until there are no more child rows for the current DataRow.

{% highlight csharp %}private JProperty GetChildRows(DataRow r)
{
	//Hack...need to create a JProperty even if there is no data
	//In your code you might want to remove this element either by a string relpace method
	JProperty o = new JProperty("ignore", "ignore");
	//If there are child rows...
	if (r.Table.ChildRelations.Count &gt; 0)
	{
		if (r.GetChildRows(r.Table.ChildRelations[0]).Length &gt; 0)
		{
			o = new JProperty(r.Table.ChildRelations[0].ChildTable.TableName,
					  new JArray(
					  from rc in r.GetChildRows(r.Table.ChildRelations[0])
					  select new JObject(
						  ConvertRowToJObject(rc),
						  GetChildRows(rc)
			  )));
		}
	}
	return o;
}
{% endhighlight %}

 [1]: http://www.www.tweetegy.com/2009/09/transform-a-dataset-to-json-using-json-net/

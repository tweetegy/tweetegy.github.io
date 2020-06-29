---
title: How to create an excel spreadsheet using Java (JExcel actually)
author: jensendarren
layout: post
permalink: /2009/07/how-to-create-an-excel-spreadsheet-using-java-jexcel-actually/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
tags:
  - excel
  - java
  - jexcel
  - spreadsheets
---
If you want to manipulate an Excel spreadsheet using .NET it is very easy to do. If your not convinced by that statement, I will show you how in a separate post. However, the purpose of this post is to create an Excel spreadsheet and populate it with values using something other than .NET. There may be many reasons why you would want to do this and mine was simply because I wanted to stick to using Linux (Ubuntu 9.10 Server).

I gave up with trying it out in Mono &#8211; some basic ADO.NET code I had written just would not run! So I turned to Java! Someone on my team recommended I try [JExcel][1] and so I did. It turned out to be a simple excercise to spike. Here is the class:

{% highlight java %}
import java.io.File;
import java.io.IOException;
import java.io.OutputStream;

import jxl.Workbook;
import jxl.write.Label;
import jxl.write.WritableSheet;
import jxl.write.WritableWorkbook;

public class JExcelSample
{
 public static void main(String args[]) {
  try
  {
   File f = new File("JExcelSample.xls");
   WritableWorkbook w = Workbook.createWorkbook(f);
   WritableSheet s = w.createSheet("Demo", 0);
   s.addCell(new Label(0, 0, "Hello World"));
   w.write();
   w.close();
  } catch (Exception e)
  {
   System.err.println(e);
  }
 }
}
{% endhighlight %}

To compile and run the above Java program you will need to download and **save jxl.jar to your /usr/lib directory** and run the following statement in your terminal:

{% highlight java %}
javac -cp "/usr/lib/jxl.jar" JExcelSample.java
java -cp "/usr/lib/jxl.jar" JExcelSample
{% endhighlight %}

That&#8217;s it! After you have run the program you will see the JExcelSample.xls file and if you open it you will see that it contains one worksheet called Demo with one entry &#8220;Hello World&#8221;. Note that you are not required to have Excel installed on your machine to do any of the above. The code runs just fine without and the spreadsheet will open in OpenOffice.org Spreadsheet by default.

 [1]: http://jexcelapi.sourceforge.net/

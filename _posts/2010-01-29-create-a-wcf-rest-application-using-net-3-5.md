---
title: Create a WCF Rest Application using .NET 3.5
author: jensendarren
layout: post
permalink: /2010/01/create-a-wcf-rest-application-using-net-3-5/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
tags:
  - .net
  - 'C#'
  - configuration
  - WCF
---
In this post, I will explain how to create a simple read only Rest service using WCF .NET 3.5 WebGet endpoints. I will also explain a little bit about the wonderful UriTemplate class.

**First create a new WCF Service Application**

Create a new WCF Service Application project. This will provide you with the standard example files for WCF that contain the *string GetData(int value)* and *CompositeType GetDataUsingDataContract(CompositeType composite)* signatures.

Now this application is not yet Rest enabled. You need to do a few more things. No doubt, of course you can turn this into your own Visual Studio project template later on if you need to.

**Rest enable the WCF Application**

Go ahead and add a reference to *System.ServiceModel.Web dll.* Now add the *WebGetAttribute* on the method you want to expose. Let's just use the supplied *GetData* method to keep things simple. However, notice that I have to change the data type of the input parameter to string since this is what the *UriTemplate* expects. This method signature in the *IService1* Interface should look something like this:

{% highlight csharp %}
[OperationContract]
[WebGet( UriTemplate = "somepath/{value}")]
string GetData(string value);
{% endhighlight %}

Then, in the web.config file that we create, we must also include the following entry (put this within the behaviors tag):

{% highlight xml %}
<endpointBehaviors>
  <behavior name="WebBehavior">
    <webHttp />
  </behavior>
</endpointBehaviors>
{% endhighlight %}

Now look within the services tag for the endpoint tag that describes the service you are building. You need to change the binding to `webHttpBinding` and add the attribute `behaviorConfiguration=WebBehavior` to this tag. The endpoint tag for your service should now look something like this:

{% highlight xml %}
<endpoint address="" binding="webHttpBinding"
contract="WcfService1.IService1"
behaviorConfiguration="WebBehavior" />
{% endhighlight %}

At this point you are ready to run the service, so hit F5. Your browser window should open to the service description page. Please then change the path following the localhost root to: ./Service1.svc/somepath/sqlserverdotnet and you should then see an XML response from the service as follows:

{% highlight xml %}
<string>You entered: tweetegy</string>
{% endhighlight %}

Now you are free to modify this service method as you want. Perhaps to connect to a database and return a collection as serialized XML for example. Obviously you can remove the .svc extension from the url by using a HttpModule. There are many examples of url re-writing on the web which demonstrate exactly how to do this.

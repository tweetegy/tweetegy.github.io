---
title: How to return content from an application running on a specific port in apache
author: jensendarren
layout: post
permalink: /2010/03/how-to-return-content-from-an-application-running-on-a-specific-port-in-apache/
image:
  -
categories:
  - Programming
tags:
  - apache
  - config
  - content
  - port
  - redirect
---
Continuing on from the [previous post][1] where I explain how to show content from a different root folder depending on the sub domain e.g. http://foo.example.com will actually return content from http://example.com/foo (without changing the URL). Today I will extend this idea to show content from a separate service running on a different port within the same server. So in this case a call to http://foo.example.com returns http://example.com:port.

As in the previous post, we assume that you already have your sub domain DNS settings correctly configured and the subdomain now points to the IP address of your server, you should do the following in Apache running on an Ubuntu server.

Open the configuration file for your site. In this example, we will use the default site.

{% highlight bash %}
nano /etc/apache2/sites-available/default
{% endhighlight %}

Add the following additional VirtualHost configuration to the end of the file (change the port number on localhost if you need to, of course!)

{% highlight xml %}
<VirtualHost *:80>
  ServerName foo.example.com
  ProxyPass / http:// localhost:3000/
  ProxyPassReverse / http:// localhost:3000/
  ProxyPreserveHost on
  <Proxy *>
   allow from all
  </Proxy>
</VirtualHost>
{% endhighlight %}

Now restart apache and your all done!

{% highlight bash %}
/etc/init.d/apache2 restart
{% endhighlight %}

 [1]: http://www.tweetegy.com/2010/04/14/how-to-subdomain-redirect-using-apache-virtual-host/

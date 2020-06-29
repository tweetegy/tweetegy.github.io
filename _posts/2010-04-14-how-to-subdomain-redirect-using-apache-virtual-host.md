---
title: How to subdomain redirect using Apache Virtual Host
author: jensendarren
layout: post
permalink: /2010/04/how-to-subdomain-redirect-using-apache-virtual-host/
image:
  -
categories:
  - Programming
tags:
  - apache
  - redirect
---
Do you want to setup a website subdomain so that, for example, http://foo.example.com will actually, without the end user realizing, return content from http://example.com/foo ? In this short, but useful blog post, I will show you how you can redirect a subdomain to a server directory without anyone (except you) knowing!

Assuming that you already have your subdomain DNS settings correctly configured and the subdomain now points to the IP address of your server, you should do the following in Apache running on an Ubuntu server.

Open the configuration file for your site. In this example, we will use the default site.

{% highlight bash %}
nano /etc/apache2/sites-available/default
{% endhighlight %}

Add the following additional VirtualHost configuration to the end of the file.

{% highlight xml %}
<VirtualHost *:80>
  ServerName foo.example.com
  DocumentRoot /var/www/foo
</VirtualHost>
{% endhighlight %}

Now restart apache and your all done!

{% highlight bash %}/etc/init.d/apache2 restart
{% endhighlight %}

If you point your browser to http://foo.example.com you will see the content from http://example.com/foo ! Note: You do, of course, have to create the folder /var/www/foo and put a index.html file there with some content like &#8220;Here be foo!&#8221; so you can test it actually works!

**Extra Bonus Script Time!**
As an added bonus, I have included a setup script that you can use to perform the above in one shot. Use it if you wish. Change foo and example.com to your domain and subdomain names.

{% highlight bash %}#!/bin/bash

SITE_CONFIG_PATH="/etc/apache2/sites-available/default"

echo "Configure default site with foo virtual host"
if ! grep -q "ServerName foo.example.com" $SITE_CONFIG_PATH
then
	echo "Configuring..."
	echo "" >> $SITE_CONFIG_PATH
	echo "# foo virtual host" >> $SITE_CONFIG_PATH
	echo "<VirtualHost *:80>" >> $SITE_CONFIG_PATH
	echo "  ServerName foo.example.com" >> $SITE_CONFIG_PATH
	echo "  DocumentRoot /var/www/foo" >> $SITE_CONFIG_PATH
	echo "</VirtualHost>" >> $SITE_CONFIG_PATH
	echo "Restart apache"
	/etc/init.d/apache2 restart
else
	echo "Already configured!"
fi
{% endhighlight %}

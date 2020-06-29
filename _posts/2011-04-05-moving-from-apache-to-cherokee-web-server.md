---
title: Moving from Apache to Cherokee Web Server
author: jensendarren
layout: post
permalink: /2011/04/moving-from-apache-to-cherokee-web-server/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
tags:
  - apache
  - bash
  - cherokee
  - migration
  - regex
  - server
  - ubuntu 10.04
---
I recently decided to move a website of mine from Apache to Cherokee. Why? Because Cherokee is fast and Google likes fast sites. Apparently, Cherokee is [the fastest web server in town][1]. I first read about Cherokee in Linux Format magazine and decided to give it a shot with my [AdFlickr][2] site which I have running on a [Linode][3] instance (Ubuntu 10.04 by the way).

## Installing Cherokee on Ubuntu 10.04

Fortunately, it is really easy to install Cherokee on Ubuntu 10.04 since version 0.99.39 is available in the software repositories. Here is basically what I run in my server.bash file to install Cherokee, PHP and MySql:

{% highlight bash %}apt-get install cherokee cherokee-doc libcherokee-mod-libssl libcherokee-mod-streaming libcherokee-mod-rrd php5-cgi php5 php5-mysql php5-xsl php5-gd php-pear curl libcurl3 libcurl3-dev php5-curl mysql-server
{% endhighlight %}

## Tunnelling into the Cherokee Admin

Cherokee includes a web admin interface for configuring the server. To get that running you need to start it up on your server and then use SSH tunnelling to view in a local browser. So on your server run the following to start the Cherokee Admin deamon:

{% highlight bash %}cherokee-admin -b &
{% endhighlight %}

Then on your local machine, setup the SSH tunnel as follows:

{% highlight bash %}ssh -L 9090:localhost:9090 mysite.com -N
{% endhighlight %}

Now, navigate to <http://localhost:9090> and log in using the username and password shown in the output on your server console. You should be greeted with the Cherokee Admin home screen!
![Cherokee Admin](/assets/cherokee-admin.png)

Configuration of Cherokee is a breeze with the use of Wizards. Just navigate to your Virtual Server Behaviour tab, click on wizards button and select the appropriate category. For PHP select Languages then Run Wizard next to PHP.

![Cherokee Wizards make configuration a breeze](/assets/cherokee-wizards.png)

## Specific issues when moving from Apache

I had two issues, one was not having installed php5-cgi which I simply added to my apt-get install list. The second issue was related to url rewriting (e.g. all the rewrite rules I had in my .htaccess file). I found that the syntax did not exactly copy accross to Cherokee. I also found that in Web Admin interface did not preserve the ordering of the rules, so I decided to edit the cherokee.conf file directly.

Here are a few examples of the rewrite rule in the Apache .htaccess file compared with the Cherokee .conf file (I have removed the vserver path from the Cherokee examples removed for clarity):

{% highlight bash %}# Example 1: In Apache .htaccess
RewriteRule ^rss/([a-zA-Z0-9_-]+)$ /rss.php?op=$1

# Example 1: In Cherokee cherokee.conf file
regex = rss/([^/]+)
show = 0
substring = rss.php?op=$1

# Example 2: In Apache .htaccess
RewriteRule ^([a-zA-Z0-9_-]+)/ad/([a-zA-Z0-9_-]+)/industry/([a-zA-Z0-9_-]+)$ dashboard.php?language=$1&module=ad&op=$2&industry=$3

# Example 2: In Cherokee cherokee.conf file
regex = ([^/]+)/ad/([^/]+)/industry/([^/]+)
show = 0
substring = dashboard.php?language=$1&module=ad&op=$2&industry=$3
{% endhighlight %}

## Production Deployment

Once I had Cherokee running perfectly in my development environment (aka my laptop), I proceeded to deploy to my production environment (aka Linode). Now, I could clone my Linode instance and deploy there and then update my DNS, but I was feeling brave and I was sure the deployment would go smoothly which it did. All I had to do was:

1.  Run server.bash to install cherokee
2.  Run a deply.bash script which (amongst other things) copies the cherokee.conf file to /etc/cherokee/
3.  Stop Apache running /etc/intit.d/apache stop
4.  Start Cherokee /etc/init.d/cherokee start

The site is now up and running on Cherokee. I hope you can join the [Cherokee crowd][4] one day too!

 [1]: http://www.cherokee-project.com/benchmarks.html
 [2]: http://www.adflickr.com
 [3]: http://www.linode.com
 [4]: http://www.cherokee-project.com/cherokee-domain-list.html

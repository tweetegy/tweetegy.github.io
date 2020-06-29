---
title: Setting up a scheduled task (cron job) to run a rake script in Ubuntu
author: jensendarren
layout: post
permalink: /2009/06/setting-up-a-scheduled-task-cron-job-to-run-a-rake-script-in-ubuntu/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
tags:
  - cron
  - crontab
  - linux
  - rake
  - schedule
  - shell
  - ubuntu
---
To schedule a task in Ubuntu you need to understand cron and crontab. Both these concepts are easy to grasp. However, nothing is always as smooth as you would want! The issues I had to resolve are:

*   Understanding the syntax of cron
*   Adding a crontab for a user
*   Creating an executable shell script which calls rake
*   Logging the output

For the syntax, I recommend checking out this Wikipedia article on [cron][1]. I will explain each of the above areas, starting with adding a crontab for a user.

**Adding a crontab for a user**

In order to edit a crontab for a user, you must type the following in the terminal:

{% highlight bash %}
sudo crontab -e
{% endhighlight %}

Note that I prefix the command with &#8220;sudo&#8221; which means it will modify the cron file for the user root. When I first tried this there was the following error: &#8220;no crontab for root&#8221;. I solved this problem with the following script which simply creates an empty file and associates that as a crontab for user root.

{% highlight bash %}touch rootcron
sudo crontab -u root rootcron
{% endhighlight %}

Now you will be able to edit the file using **sudo crontab -e** and it will work just fine.

**Creating an executable shell script which calls rake**

Let&#8217;s say you have a rake script in /var/www/someproject/rakefile. You can execute the rake script using the following script:

{% highlight bash %}#!/bin/sh
date
echo "Executing rake"
cd /var/www/someproject/
rake
{% endhighlight %}

You must make this script an executable before cron can start it up. Here&#8217;s how:

{% highlight bash %}sudo chmod +x runrake.sh
{% endhighlight %}

You can now execute this every 24hrs at midnight (for example) by adding the following line to your cron. Note that the full path to the script is included and there is a &#8220;./&#8221; before the name of the script, which is how executable files can be run in the terminal. Next issue. Sometimes the cron job would run but not complete or just did not appear to run at all. How can I get more information about what is happening? Let&#8217;s check out logging!

{% highlight bash %}0 0 * * * sudo /var/www/someproject/./runrake.sh
{% endhighlight %}

**Logging the output**

It turns out to be really straight forward to log the output of a cron job. Simply use the redirect output syntax to send the standard output to a file. Use single > to create a new file each time the cron is run or double >> to append to your log file. You can put the file anywhere you like, however, it is recommended to put inside the standard log folder /var/log. So now the line in the cron file looks as follows:

{% highlight bash %}0 0 * * * sudo /var/www/someproject/./runrake.sh > /var/log/runrake.log
{% endhighlight %}

 [1]: http://en.wikipedia.org/wiki/Cron

---
title: Install WordPress on Ubuntu 10.04 Slicehost
author: jensendarren
layout: post
permalink: /2010/12/install-wordpress-on-ubuntu-10-04-slicehost/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
tags:
  - bash
  - server
  - slicehost
  - ubuntu
  - wordpress
---
Here is a quick step by step for setting up a new Slicehost slice and installing WordPress on that slice. In this example we will use a Ubuntu 10.04 (Lucid Lynx) 32-bit 256 MB slice. The first part is a condensed version of <a title="Ubuntu Lucid Setup on Slicehost" href="http://articles.slicehost.com/2010/4/30/ubuntu-lucid-setup-part-1" target="_self">these instructions.</a> The second part introduces a bash script that will install and configure a LAMP stack in order to run WordPress.

## Part 1: Fire up and configure your new Slice!

Goto [slicehost][1], sign up, and pick your slice by selecting &#8220;Ubuntu 10.04 LTS (lucid) 32-bit&#8221; from the Linux Distribution drop down menu. Wait for the slice to fire up and then when it is marked *active* in your control panel you can ssh into the slice using the root username and password.

{% highlight bash %}ssh root@184.106.xxx.xxx{% endhighlight %}

First change the password of root

{% highlight bash %}passwd{% endhighlight %}

Add a new user and group. Follow the instructions on the screen.

{% highlight bash %}adduser demouser{% endhighlight %}

Open visudo

{% highlight bash %}visudo{% endhighlight %}

Add this line to the bottom of the file:

{% highlight bash %}demouser  ALL=(ALL) ALL{% endhighlight %}

On your LOCAL machine generate an ssh key

{% highlight bash %}mkdir ~/.ssh
ssh-keygen -t rsa
{% endhighlight %}

Copy this key to your new slicehost slice using scp on your LOCAL machine

{% highlight bash %}scp ~/.ssh/id_rsa.pub root@184.106.xxx.xxx:
{% endhighlight %}

Back on the SERVER &#8211; your new Slicehost slice, run the following commands. This simply moves your local machine public key to the authorized_keys file for demouser.

{% highlight bash %}mkdir /home/demouser/.ssh
mv id_rsa.pub /home/demouser/.ssh/authorized_keys
chown -R demouser:demouser /home/demouser/.ssh
chmod 700 /home/demouser/.ssh
chmod 600 /home/demouser/.ssh/authorized_keys
{% endhighlight %}

Make ssh more secure by editing `/etc/ssh/sshd_config` and changing the following three properties:

*   Port 30000 <\--- change to a port of your choosing
*   PermitRootLogin no
*   PasswordAuthentication no

{% highlight bash %}nano /etc/ssh/sshd_config
{% endhighlight %}

Load custom rules into your server iptables. The easiest thing to do is copy paste the [Slicehost iptables rules][2] to a new file `/etc/iptables.up.rules` and then load these rules. Remember to open the port that you selected in the previous step for ssh.

{% highlight bash %}nano /etc/iptables.up.rules
{% endhighlight %}

Now run the following to update rules from the new file

{% highlight bash %}/sbin/iptables-restore &lt; /etc/iptables.up.rules
{% endhighlight %}

You want to ensure that these rules are loaded on every instance boot. So create a new file `/etc/network/if-pre-up.d/iptables` and add the following script into this file:

{% highlight bash %}#!/bin/sh
/sbin/iptables-restore &lt; /etc/iptables.up.rules
{% endhighlight %}

Run this to make the file executable

{% highlight bash %}chmod +x /etc/network/if-pre-up.d/iptables
{% endhighlight %}

Reload SSHD

{% highlight bash %}/etc/init.d/ssh reload
{% endhighlight %}

Now you can try logging in as the new user (demouser in this exampe) from your LOCAL machine

{% highlight bash %}ssh -p 30000 demouser@184.106.xxx.xxx
{% endhighlight %}

If all goes well, you can log out of root at this point and continue to use demouser going forward. If you do get locked out, Slicehost have a browser based console, which you can use to attempt to fix any issues with SSH.

In Part 2 we will prepare a bash script for installing WordPress.

 [1]: http://slicehost.com
 [2]: http://articles.slicehost.com/assets/2007/9/4/iptables.txt

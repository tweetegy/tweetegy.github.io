---
title: Hudson setup to execute scripts with root privileges (safely)
author: jensendarren
layout: post
permalink: /2010/05/hudson-setup-to-execute-scripts-with-root-privileges-safely/
image:
  -
categories:
  - Programming
tags:
  - execution
  - hudson
  - linux
  - root
  - script
---
There is a lot out there on the web about how to setup and install [Hudson][1]. In fact that process is very easy so I will just provide a script to explain that (see the end of this post). What I found to be an interesting challenge is getting Hudson to execute my scripts with root privileges&#8230;

The first time I execute the script (without sudo), my job fails and I am informed that the user hudson does not have permission to execute apt-get. So naturally, I prefix the script with sudo and my job fails again, this time with an incorrect password. So how to get Hudson to execute my script with root privileges?. After a lot of searching I managed to <a title="Blog Flex" href="http://fbflex.wordpress.com/2008/05/02/howto-setting-up-a-continous-integration-server-for-grails-with-hudson-on-vmware/" target="_self">find one blog post</a> which mentions adding the user (in my case user hudson) to the sudoers file and giving the user full access without being promted for a password. Using visudo, I added this line to my sudoers file:

{% highlight bash %}hudson  ALL=(ALL) NOPASSWD:ALL
{% endhighlight %}

However, I was most concerned about giving hudson the rights to do what it wanted without requiring a password as this leaves an obvious security hole. So I <a title="Serverfault Question Husdon Root" href="http://serverfault.com/questions/141439/execute-build-task-in-hudson-with-root-privilages" target="_self">posted my concern on ServerFault.com</a> and fortunately I was informed that it is possible to lock down the commands and the parameters that can be used by a user without requiring a password. So I changed my entry in sudoers file to this:

{% highlight bash %}hudson  ALL=(ALL) NOPASSWD:/var/scripts/the-script-I-want-to-run.bash
{% endhighlight %}

Now all hudson can do (without being prompted for a password) is to execute sudo /var/scripts/the-script-I-want-to-run.bash. This makes be feel safe and my build works a treat each and every time too! I thought I would share this since I could not find a clear solution to this issue.

By the way, if you want to install hudson on your linux server, here is the script I prepared earlier!

{% highlight bash %}#Install Hudson
#NOTE: You MUST manually add hudson user to the sudoers account using visudo,
#specifying all the scripts that you want hudson to be able to run without requiring a root password, for example:
#hudson  ALL=(ALL) NOPASSWD:/var/scripts/the-script-I-want-to-run.bash
wget -O - http:// hudson-ci.org/debian/hudson-ci.org.key | sudo apt-key add -
if ! grep -q "http:// hudson-ci.org/debian binary/" /etc/apt/sources.list ; then
    echo "deb http:// hudson-ci.org/debian binary/" >> /etc/apt/sources.list
fi
#NOTE: On a fresh install, you will have to run this script twice since the first time the script will not know about sources.list update
apt-get install hudson
#Install Hudson Plugins
if test ! -f /var/lib/hudson/plugins/mercurial.hpi ; then
    wget -P /var/lib/hudson/plugins http:// hudson-ci.org/download/plugins/mercurial/1.28/mercurial.hpi
    chown hudson:nogroup /var/lib/hudson/plugins/mercurial.hpi
fi
{% endhighlight %}

Additionally, if you want Hudson to be accessible at http://hudson.yourserver.com while keeping Hudson running on port 8080 and you are running Apache then run the following script!

{% highlight bash %}SITE_CONFIG_PATH="/etc/apache2/sites-available/default"

if ! grep -q "ServerName hudson.yourserver.com" $SITE_CONFIG_PATH
then
	echo "Configuring..."
	echo "" >> $SITE_CONFIG_PATH
	echo "# Hudson virtual host" >> $SITE_CONFIG_PATH
	echo "&lt;VirtualHost *:80>" >> $SITE_CONFIG_PATH
	echo "  ServerName hudson.yourserver.com" >> $SITE_CONFIG_PATH
	echo "  ProxyPass / http:// localhost:8080/" >> $SITE_CONFIG_PATH
	echo "  ProxyPassReverse / http:// localhost:8080/" >> $SITE_CONFIG_PATH
	echo "  ProxyPreserveHost on" >> $SITE_CONFIG_PATH
	echo "  &lt;Proxy *>" >> $SITE_CONFIG_PATH
	echo "   allow from all" >> $SITE_CONFIG_PATH
	echo "  &lt;/Proxy>" >> $SITE_CONFIG_PATH
	echo "&lt;/VirtualHost>" >> $SITE_CONFIG_PATH

	#Create symlink to proxy module
	ln -s ../mods-available/proxy.load /etc/apache2/mods-enabled/proxy.load
	ln -s ../mods-available/proxy.conf /etc/apache2/mods-enabled/proxy.conf
	ln -s ../mods-available/proxy_http.load /etc/apache2/mods-enabled/proxy_http.load

	echo "Restarting apache"

	#Restart Apache
	/etc/init.d/apache2 restart
else
	echo "Already configured!"
fi
{% endhighlight %}

 [1]: http://hudson-ci.org/

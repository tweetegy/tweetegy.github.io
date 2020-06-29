---
title: Install WordPress on Ubuntu 10.04 Slicehost (Part 2)
author: jensendarren
layout: post
permalink: /2010/12/install-wordpress-on-ubuntu-10-04-slicehost-part-2/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
  - MySql
tags:
  - bash
  - mysql
  - wordpress
---
In [Part 1][1] we setup a new virtual host instance using Slicehost. The steps can be pretty much the same for any new Ubuntu server. It&#8217;s just that in this case we are using 10.04 on Slicehost.

Once you have the slice up and running then you probably want to do an update and then run a simple script to install and configure WordPress. Here is a nice script that will install WordPress and add a couple of plugins as a bonus!

Note there is a second SQL script to setup the db that is required. It is shown after the first script below. You&#8217;ll probably want to replace YOUR-USERNAME and YOUR-PASSWORD with your own values! Note also that the name of the MySQL database is *wordpress* (which you can also change if you like)

{% highlight bash %}
#!/bin/bash

#############################################################
## INSTALLS AND CONFIGURES WORDPRESS ON SERVER             ##
##                                                         ##
#############################################################

#Packages
apt-get -y install mysql-server php5 libapache2-mod-php5 php5-xsl php5-gd php-pear libapache2-mod-auth-mysql php5-mysql

#Install WordPress
if [ ! -d /var/www/wp-content ]; then
	echo "#############################"
	echo "## START INSTALL WORDPRESS ##"
	echo "#############################"

	SCRIPT_DIR=`dirname "$0"`
	WORDPRESS_PATH="/var/www"

	mysql -u root --password="YOUR-PASSWORD" &lt; $SCRIPT_DIR/setupdb.sql

	#Set up wordpress directories
	[ ! -d $WORDPRESS_PATH ] &#038;&#038; mkdir $WORDPRESS_PATH

	#Download and install wordpress. Here is an alternative: https://help.ubuntu.com/community/WordPress
	mkdir $SCRIPT_DIR/downloads
	wget http://wordpress.org/latest.tar.gz -O $SCRIPT_DIR/downloads/latest.tar.gz
	tar -xzvf $SCRIPT_DIR/downloads/latest.tar.gz --directory=$SCRIPT_DIR/downloads/
	mv $SCRIPT_DIR/downloads/wordpress/wp-config-sample.php $SCRIPT_DIR/downloads/wordpress/wp-config.php
	mv $SCRIPT_DIR/downloads/wordpress/* $WORDPRESS_PATH

	#Remove the default index.html file
	rm /var/www/index.html

	#Add our database settings to wp-config.php
	sed -i 's/database_name_here/wordpress/g' "$WORDPRESS_PATH/wp-config.php"
	sed -i 's/username_here/YOUR-USERNAME/g' "$WORDPRESS_PATH/wp-config.php"
	sed -i 's/password_here/YOUR-PASSWORD/g' "$WORDPRESS_PATH/wp-config.php"

	#Install our theme
	unzip $SCRIPT_DIR/theme.zip -d $WORDPRESS_PATH/wp-content/themes/

	#Install some plugins (G-Lock Opt-in and Google Sitemaps)
	wget http://downloads.wordpress.org/plugin/google-sitemap-generator.3.2.3.zip
	wget http://downloads.wordpress.org/plugin/g-lock-double-opt-in-manager.zip
	unzip google-sitemap-generator.3.2.3.zip -d /var/www/wp-content/plugins/
	unzip g-lock-double-opt-in-manager.zip -d /var/www/wp-content/plugins/

	#Enable mod_rewrite
	cd /etc/apache2/mods-enabled
	ln -s ../mods-available/rewrite.load rewrite.load

	#Clean up
	rm -rf $SCRIPT_DIR/downloads

	echo "###########################"
	echo "## END INSTALL WORDPRESS ##"
	echo "###########################"
fi
{% endhighlight %}

Here is the setupdb.sql script that is used in the above installation script.

{% highlight sql %}
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "YOUR-USERNAME"@"localhost" IDENTIFIED BY "YOUR-PASSWORD";
FLUSH PRIVILEGES;
{% endhighlight %}

What about Apache configuration I hear you cry! Let's put that in Part 3 shall we?!

 [1]: /2010/12/install-wordpress-on-ubuntu-10-04-slicehost/

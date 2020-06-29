---
title: command to ftp in linux bash script
author: jensendarren
layout: post
permalink: /2010/11/command-to-ftp-in-linux-bash-script/
image:
  -
categories:
  - Bash Scriping
tags:
  - bash
  - ftp
  - linux
---
Command line ftp scripts are often unappreciated. Often, when a requirement to ftp files arises it makes sense to write a ftp command file which automates this. The question is what is the best approach? Out of the box, most Linux and Unix ftp commands are limited. In particular, recursive folder creation on the ftp server is not possible with the basic linux / unix ftp tool. It lacks a command in ftp that offers this recursive functionality. So what is the solution if you require a bash script ftp for upload of files and folders?

I recommend you download ,install and use [ncftp][1]. This is a far superior option compared with the basic set of ftp commands that comes with the installed ftp program with linux.

It would not be fair to leave without providing an example so here is one! Imagine that you have a set of nested folders, some which are empty and some which contain files. You want to ftp the entire folder structure (including the files, of course!) to a remote ftp server. Here is a ftp bash script to do just that (replace with your settings where appropriate).

{% highlight ruby %}
FTP_HOST=ftp.yoursite.com
FTP_LOGIN=yourusername
FTP_PASSWORD=yourpassword

ncftp << EOF
open -u $FTP_LOGIN -p $FTP_PASSWORD $FTP_HOST
cd somefolder/onyourserver
lcd "somefolder/onyourclient"
put -R *
bye
EOF
{% endhighlight %}

Save the above script in a file named **ftp-example.sh** and execute it in your terminal as you would any shell script. The key command in this example which creates all the folders recursively is **put -R ***. The rest of the example is pretty straight forward.

 [1]: http://www.ncftp.com

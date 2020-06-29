---
title: Step by step guide to setting up a client server workstation on Ubuntu and VirtualBox
author: jensendarren
layout: post
permalink: /2010/03/step-by-step-guide-to-setting-up-a-client-server-workstation-on-ubuntu-and-virtualbox/
image:
  - 
categories:
  - Programming
tags:
  - client-server
  - dev
  - software development
  - virtualbox
---
I work on a number of different software projects. During the week I work on project called <a title="Yoolk" href="http://www.yoolk.com" target="_blank">Yoolk</a> and during the evenings and weekends I might work on <a title="adflickr global ad archive database" href="http://www.adflickr.com" target="_blank">AdFlickr</a> or some other new project! Either of these main projects might also have sub-projects. The cleanest way to separate all of these projects is to develop the software using a guest OS running in VirtualBox. It makes sense, of course, that the guest OS matches your production environment as much as possible. Some of the benefits of doing this are:

*   Complete separation of all your project dependencies. Change a setting on the guest development server for one project has absolutely no affect on your other projects.
*   Allows much more freedom to experiment with different dependencies, try out new software and generally just screw around without worrying about having to clean up your mess after (just take snapshots or re-install your guest OS!).
*   During development your are running your application in an environment which (at OS level at least) should be exactly the same as production. Therefore, this minimizes surprises during deployment.
*   Allows you to keep your client OS the way you want it so that your development experience is just the way you like it. For example, installing different text editors, music libraries, video libraries, browsers etc etc without worrying that it will affect the application your building.

While the benefits are great, there is a little work to be done to set this up initially (but it&#8217;s not too much). The longest step is installing the guest OS on VirtualBox. Obviously this depends on which guest OS you install. I use Ubuntu 9.04 Server Edition. By the way, my host OS is currently Ubuntu 9.10 but I know that this also works on Ubuntu 9.04 (since I used to run that before).

Below is a step by step guide to setting this up. Notice that you will be alternating between the client (your desktop OS) and the server (the guest OS in VirtualBox):

1.  **client**: install server on virtualbox (make the default user the same name as client user name)
2.  **client**: configure virtualbox to use bridged networking and start server
3.  **server**: sudo apt-get update
4.  **server**: sudo apt-get install openssh-server
5.  **server**: ifconfig (and note the ip address)
6.  **client**: ssh into the server using the IP address you recorded in the previous step
7.  **server**: mkdir .ssh
8.  **client**: ssh-keygen (don&#8217;t enter any values, press return three times, yes passwords should be blank)
9.  **client**: cat ~/.ssh/id_rsa.pub &#8211; copy the output to the clipboard (very carefully, no pre/trailing white space)
10. **server**: touch .ssh/authorized_keys
11. **server**: sudo nano .ssh/authorized_keys &#8211; paste clipboard contents
12. **client**: sudo nano /etc/hosts &#8211; add line: 192.168.???.??? myappname.dev www.myappname.dev (get ip from ifconfig)
13. **client**: sudo /etc/init.d/networking restart (refresh new data in /etc/hosts)
14. **client**: In nautilus go to ssh://myappname.dev then add the location to bookmarks
15. **client**: In your web browser, go to http://myappname.dev and add the location to bookmarks (you will not get a response unless you have a web server running your application on your guest OS, of course!).

After you have completed the above steps when logging in via ssh you shouldn&#8217;t be prompted for a password! Note, if IP address of the server changes, you should edit /etc/hosts to reflect that change.
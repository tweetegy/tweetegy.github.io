---
title: Clone a VirtualBox VDI using VBoxManage on the command line
author: jensendarren
layout: post
permalink: /2010/12/clone-a-virtualbox-vdi-using-vboxmanage-on-the-command-line/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
tags:
  - bash
  - clone
  - productivity
  - vdi
  - virtualbox
---
I run into this situation all the time. I want to try out a new software (usually it&#8217;s a database of somesort) and I want to try it on a clean server installation. I don&#8217;t want to mess up my client or an existing server or have to install the OS fresh each time, so I turn to VirtualBox cloning via the command line.

Let&#8217;s spec out what we want the script to do:

1.  Clone a clean install VDI that exists in a known location (typically ~/.VirtualBox/HardDisks/)
2.  Create a new virtual machine
3.  Attach the new VDI to that machine
4.  Setup a standard configuration for boot order, bridged networking etc
5.  Fire up the instance

Here is the final script. Note it includes a param to set the name of the new machine (not tested with spaces in the name yet). Note also, I have set my network to use wlan0. You might want to use a different network adapter.

{% highlight bash %}#!/bin/bash
declare EXISTING_VDI="Ubuntu_10.04_Clean_Install.vdi"
declare CLONE_NAME="Ubuntu_10.04_Clean_Install_Clone"

if [ "$#" -eq 1 ] ; then
    CLONE_NAME="Ubuntu_10.04_"$@
fi

echo "Cloning from: " $EXISTING_VDI
echo "The new VDI name: " $CLONE_NAME

VBoxManage clonevdi ~/.VirtualBox/HardDisks/$EXISTING_VDI ~/.VirtualBox/HardDisks/$CLONE_NAME.vdi
VBoxManage createvm -name $CLONE_NAME -register
VBoxManage modifyvm $CLONE_NAME  --hda ~/.VirtualBox/HardDisks/$CLONE_NAME.vdi
VBoxManage modifyvm $CLONE_NAME  --nic1 bridged
VBoxManage modifyvm $CLONE_NAME  --bridgeadapter1 wlan0
VBoxManage modifyvm $CLONE_NAME  --boot1 disk
VBoxManage startvm $CLONE_NAME
{% endhighlight %}

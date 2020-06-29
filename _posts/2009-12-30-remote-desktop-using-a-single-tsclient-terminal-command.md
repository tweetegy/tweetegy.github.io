---
title: Remote desktop using a single tsclient terminal command
author: jensendarren
layout: post
permalink: /2009/12/remote-desktop-using-a-single-tsclient-terminal-command/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
tags:
  - alias
  - bash
  - bashrc
  - remote
  - remote desktop
  - server
  - terminal
  - tsclient
  - ubuntu
---
If you use Ubuntu and have to work on remote computers that require using a GUI (like Windows machines for example), then you will find this following tip very useful! Firstly let me explain what you will have; after just a few minutes of minor configuration, you will be able to remote desktop into a Windows (or other) machine with just one swift command in the terminal! Say for example, you have a remote machine called &#8220;jungle&#8221; then wouldn&#8217;t it be nice if you could remote to it using the following single command?

{% highlight bash %}remotejungle
{% endhighlight %}

I Thought so! OK, so here&#8217;s how! Firstly create a folder called .tsclient under your home directory (note the period before the name of the folder so that it is kept hidden by default).

{% highlight bash %}cd
mkdir .tsclient
{% endhighlight %}

Create a file (if not already) called .bash_aliases and save it under your home directory:

{% highlight bash %}cd
touch .bash_aiases
{% endhighlight %}

Now edit that file and add the alias for remoting into your remote machine, like so:

{% highlight bash %}alias remotejungle='tsclient -x ~/.tsclient/remotejungle.rdp'
{% endhighlight %}

Note that this sets up the alias &#8220;remotejungle&#8221; so that it executes the command **&#8220;tsclient -x ~/.tsclient/remotejungle.rdp&#8221;**. As you can see it expects to find a file **remotejungle.rdp** in the .tsclients folder. In order to create this file you need to use the &#8220;Terminal Server Client&#8221; GUI application to setup your desired session settings for display screen size, color depth, remote sound, keyboard and so on and then click the &#8220;Save As&#8221; button and save the file as remotejungle.rdp to ~/.tsclient/.

The final step is to modify the .bashrc file so that it picks up the alias list. Open up the .bashrc file in an editor and uncomment the following lines:

{% highlight bash %}if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
{% endhighlight %}

You&#8217;ll probably have to reload the .bashrc file again by opening a new terminal or executing the following command should also reload the .bashrc file:

{% highlight bash %}source .bashrc
{% endhighlight %}

Now your good to go! Just try to remote into your computer using the alias that you setup during this tutorial and you will never be happier to remote into Windows from Ubuntu!

![Ubuntu - Terminal Server Client](/assets/terminal-server-client-jungle.png)

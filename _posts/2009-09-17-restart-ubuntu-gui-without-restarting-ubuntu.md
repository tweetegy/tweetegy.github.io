---
title: Restart Ubuntu GUI without restarting Ubuntu
author: jensendarren
layout: post
permalink: /2009/09/restart-ubuntu-gui-without-restarting-ubuntu/
image:
  -
seo_follow:
  - 'false'
categories:
  - Bash Scriping
  - Programming
tags:
  - bash
  - linux
  - tty
  - ubuntu
---
Here is a little trick I learned recently from [Tropical Ice Cube][1]. I am fairly new to Ubuntu and so all of what I heard was very new to me. So here is how you can &#8220;get out&#8221; of the graphical user interface, and restart it without having to reboot your machine.

To start off with, press **CTRL+ALT+F2**
At this point your entire screen will become a terminal window, so to speak.
**Note: You can go back to your GUI by pressing: ALT+F7**

Now, back in the terminal, you can find the Ubuntu GUI process by running the following command. *Note: once you see the results, you might want to scroll up or down the screen, in which case you can hold SHIFT and page-up or page-down keys to do that.*

{% highlight bash %}ps -A
{% endhighlight %}

Look for the process Xorg and gdm in the list &#8211; these are running the GUI. Of couse, you can also pipe the output of ps -A to grep to filter out Xorg and gdm processes as follows:

{% highlight bash %}ps -A | grep Xorg
ps -A | grep gdm
{% endhighlight %}

You will probably notice that the output for Xorg is something like below. Note entry **tty7**. This stands for teletype7 and harks back to the old unix days. For more information about tty, see [Ubuntu Forums post on tty and pty.][2]. You&#8217;ll see that It is no coincidence that CTLT+ALT+F7 brings you back to the GUI!

{% highlight bash %}2899 tty7     00:59:39 Xorg
{% endhighlight %}

Now run the killall command which terminates the specific processes as well as their children:

{% highlight bash %}killall Xorg gdm
{% endhighlight %}

If you hit ALT+F7 now, there is no GUI! Where has it gone? Well you killed that process and in order to start it up again, run the following in the terminal:

{% highlight bash %}sudo gdm
{% endhighlight %}

This will launch the GUI and take you back to the Ubuntu login screen. You will have to open all your GUI based applications again, of course. One thing also to note is that the GUI might start on a different tty. To check this type the following command in the terminal:

{% highlight bash %}who
{% endhighlight %}

In my case, I see the following output. This tells me that there are two tty sessions running; one on tty2 (which is the terminal application I just used to restart gdm) and tty9 which is where the new gdm instance started. So now this basically means to get back to the GUI I need to hit **CTRL+ALT+F9**.

{% highlight bash %}darren tty2         2009-09-17 10:29
darren tty9         2009-09-17 10:50 (:0)
{% endhighlight %}

 [1]: http://tropicalicecube.net
 [2]: http://ubuntuforums.org/showthread.php?t=833765

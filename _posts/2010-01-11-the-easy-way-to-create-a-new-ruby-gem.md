---
title: The easy way to create a new Ruby Gem!
author: jensendarren
layout: post
permalink: /2010/01/the-easy-way-to-create-a-new-ruby-gem/
image:
  -
seo_follow:
  - 'false'
categories:
  - Ruby Server Programming
tags:
  - api gems
  - gems
  - jewler
  - ruby
---
If you want to make a Ruby Gem and want to do it in a very easy way then here&#8217;s is one very good option &#8211; use <a title="Jeweler Gem" href="http://github.com/technicalpickles/jeweler" target="_self">Jeweler</a>! Thanks to <a title="RailsTips" href="http://railstips.org/" target="_self">John Nunemaker at RailsTips</a> for pointing me in the right direction with his post <a title="Building API Wrapping Gems" href="http://railstips.org/blog/archives/2009/03/25/building-api-wrapping-gems-could-not-get-much-easier/" target="_self">Building API Wrapping Gems</a>

1.  **Install git core** {% highlight bash %}apt-get install git-core{% endhighlight %}

2.  **Install jeweler gem** {% highlight bash %}gem install jeweler{% endhighlight %}

3.  **Now add the following line to your bashrc file to ensure that you always use spec (that if you *do* use spec). Otherwise the default is shoulda and if your happy using that then you can skip this part.** {% highlight bash %}export JEWLER_OPTS = "--spec"{% endhighlight %}

4.  **Setup your global username and email for git** {% highlight bash %}git config --global user.name "Your Company Name"
git config --global user.email "yourname@yourcompanyname.com"{% endhighlight %}

5.  **Script your new gem using jeweler** {% highlight bash %}jeweler your_shiny_new_gem{% endhighlight %}

6.  **Set your init version**. **I do this so that I can easily generate the gemspec later.** {% highlight bash %}rake version:write MAJOR=0 MINOR=1 PATCH=0{% endhighlight %}

7.  **Modify the new gem rake file to add the meta info for your project**
    [source lang="ruby"]
    begin
    require &#8216;jeweler&#8217;
    Jeweler::Tasks.new do |gem|
    gem.name = &#8220;your\_shiny\_new_gem&#8221;
    gem.summary = %Q{A simple gem to access the Shiny API}
    gem.description = %Q{Use this gem to access our Shiny API !!}
    gem.email = &#8220;yourname@yourcompanyname.com&#8221;
    gem.homepage = &#8220;http://bitbucket.org/yourrepo/your\_shiny\_new_gem&#8221;
    gem.authors = ["Your Company Name"]
    gem.add\_development\_dependency &#8220;rspec&#8221;, &#8220;>= 1.2.9&#8243;
    \# gem is a Gem::Specification&#8230; see http://www.rubygems.org/read/chapter/20 for additional settings
    end
    Jeweler::GemcutterTasks.new
    rescue LoadError
    puts &#8220;Jeweler (or a dependency) not available. Install it with: gem install jeweler&#8221;
    end
8.  **Generate the gemspec** {% highlight bash %}rake gemspec{% endhighlight %}

9.  **Write the code that goes into your gem. This is where your own imagination comes in! See the <a title="Google Weather API Gem" href="http://github.com/jnunemaker/google-weather" target="_self">Google Weather API Gem</a> for an example. When your gem is finished (including all your tests passing!) move on to building and testing the installation of the Gem. **** ** {% highlight bash %}code.write!{% endhighlight %}

10. **Build the gem** {% highlight bash %}rake build{% endhighlight %}

11. **Install the gem** {% highlight bash %}rake install{% endhighlight %}

You can deploy to github (the easiest solution when using Jeweler). However, you can still deploy to other repositories like Bitbucket for example. Then, as recommended by the author of Jeweler, <a title="Josh Nichols" href="http://technicalpickles.com/about/" target="_self">Josh Nichols</a>, have a delicious beverage!

Happy Gem stone cutting!

---
title: Using Railtie and Rails Engine in Gems
author: jensendarren
layout: post
permalink: /2012/09/using-railtie-and-rails-engine-in-gems/
image:
  -
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
tags:
  - asset pipeline
  - engine
  - engines
  - gems
  - rails
  - rails engines
  - railtie
  - railties
  - ruby
---
A (private) Gem I made recently required some code modifications that required digging into Railties and Rails Engines a little. The requirements were to inject some Javascript variables into the view on each request and to include a static Javascript file in the Asset Pipeline of the consuming Rails App (I know that this Gem will be used in Rails 3.x).

## Problem 1: Inject Javascript vars into the view on each render using a Gem

So for the first problem I naturally chose to extend the Rails middleware stack. For problems like this I tend to work with a test Rails application to try things out first before moving the solution into a Gem. The solution for this problem (if putting the ruby file directly in the Rails lib directory) is as follows:

{% highlight ruby linenos %}
require 'rack'

  class JSVars

    def initialize(app, options = {})
      @app = app
    end

    def call(env)
      status, headers, response = @app.call(env)

      if status != 301 && response.respond_to?(:request)
        response_string = inject_vars(response, headers)

        response = Rack::Response.new(response_string, status, headers)
      else
        Rack::Response.new(response, status, headers)
      end
    end

    ## inject_vars implementation not shown
    ## all it does is add a <script> tag containing inline Javascript within the <head> tag of the html response string
end
{% endhighlight %}

In addition to this code living in `lib/js_vars.rb` it was necessary to add a line to configure the middleware of the application to use this, as follows:

{% highlight ruby linenos %}config.middleware.use "JSVars"
{% endhighlight %}

This solution worked well directly inside the Rails app but I needed to share this functionality with numerous Rails apps so I needed it in a Gem. How can I configure middleware in a Rails app from my Gem? The answer is with a Railtie!

## Solution 1: Use Railties!

Using Railties the solution is fortunately very simple. After moving the js_vars.rb file into the gems lib directory and removing the middleware configuration from the app, I added the following Railtie class to my Gem as follows:

{% highlight ruby linenos %}
class Railtie < Rails::Railtie
  initializer "my_gem.insert_middleware" do |app|
    app.config.middleware.use "MyGem::JSVars"
  end
end
{% endhighlight %}

After adding the necessary `require` statements in my Gem, that was it! When I restarted my Rails app, the same functionality was working - the variables were being injected into the page as expected! Great! Now onto the next problem!

## Problem 2: Dynamically add assets to the Rails Asset Pipeline from a Gem

I had a Javascript file that made use of these variables and I needed that to also reside within the Gem, but how to get that Javascript into the host Rails Asset Pipeline? The answer is use Rails Engines!

{% highlight ruby linenos %}
class Engine < Rails::Engine
end
{% endhighlight %}

After adding the necessary `require` statements for this Engine in my Gem, you guessed it, everything worked (again)! By simply including this class in the Gem it means that the Rails app the Gem is loaded into will search for assets inside the Gem! So if we add a .js file to `app -> assets -> javascripts` path (in the Gem) then this file can be referenced (and found!) in the Rails app as follows:

{% highlight ruby linenos %}
//= require my_gems_javascript
{% endhighlight %}

## Conclusion

Rails 3.x brings us a whole set of wonderful and powerful tools: **Engines, Railties, Middleware and the Asset Pipeline** (as well as many other goodies!). If we want to split off and reuse functionality we can use Gems together with a mixture of these new tools. In this example, I used:

*   Railties to extend Middleware
*   Engines to extend the Asset Pipeline

**Of course this is only a small subset of the power of these tools** and I highly recommend reading more about this subject.

## Recommended Further Reading / Watching

*   [Writing Rails Engines Getting Started][1]
*   [Create your own Rails 3 Engine][2]
*   [Writing a Rails Engine (YouTube video)][3]
*   [Engines (Vimeo video by Ryan Bigg)][4]
*   [Railties][5]

 [1]: http://www.stubbleblog.com/index.php/2011/04/writing-rails-engines-getting-started/
 [2]: http://gregmoreno.wordpress.com/2012/05/29/create-your-own-rails-3-engine/
 [3]: http://www.youtube.com/watch?v=Rvxcc46fox0&feature=em-uploademail
 [4]: http://vimeo.com/41952172
 [5]: http://www.igvita.com/2010/08/04/rails-3-internals-railtie-creating-plugins/

---
title: 'Dissecting thesmallestrailsapp.com (Smallest Rails App: line 4)'
author: jensendarren
layout: post
permalink: /2013/01/dissecting-thesmallestrailsapp-com-smallest-rails-app-line-4/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - block
  - config
  - proc
  - rails
  - ruby
  - secret_token
  - smallesrailsapp
  - to_s
---
In this post, I will carry on from where I left off in the previous post and continue my dissection of the Smallest Rails app! This takes us onto line 4 of the code. Line 4 looks like this:

{% highlight ruby linenos %}
config.secret_token = routes.append {
{% endhighlight %}

This shows the Rails config `secret_token` being set to return value of routes.append. What's going on here then and why are we even doing this?

## Rails config.secret_token

In order for Rails to process requests `config.secret_token` must be set to a random string of at least 30 chars that is difficult (near impossible) to guess. The purpose of the secret key is to verify the integrity of signed cookies :) without it Rails will fail to process requests with the following 500 error:
`ArgumentError (A secret is required to generate an integrity hash for cookie session data. Use config.secret_token = "some secret phrase of at least 30 characters" in config/initializers/secret_token.rb)`

This is a generic error message since the `secret_token` in the Smallest Rails App is set in config.ru file. Notice the comment *&#8220;some secret phrase of at least 30 characters&#8221;* which means the phrase can be any string (30 chars or more in length). In the Smallest Rails App this is set to the string value of the `routes.append` block (notice the call to `to_s` in line 14). On my machine this returns the following *string*:
`[#<Proc:0x007fa5842838c8@/thesmallestrailsapp.com/config.ru:5>]`

Note that in a conventional Rails App the `secret_token` will be generated automatically and will look something like this `0eb7e863b78c7bdc52376bc0f24b2c634fa8e1ca83e2d7b695da86b44e80194.....`.

If you need to regenerate the secret_key you can run `SecureRandom.hex(64)` in the console and copy the output to your initializer file or run `rake secret` which does this same thing without having to log into the Rails console.

{% highlight ruby linenos %}rake secret
    Generate a cryptographically secure secret key (this is typically used to generate a secret for cookie sessions).
{% endhighlight %}

**Warning**: if you change this key you will need to restart your application. Additionally, existing signed cookies will become invalid effectively throwing currently logged in or in session users off your system! It&#8217;s probably not necessary to ever change this key unless you think your applications security has been compromised.

## Secure your Secret Token

A quick note about securing your secret token. If your project is open source or you have a number of developers working on a private repo then you *should not* commit the actual production secret token into your source control repository! Since the secret token can be used to generate signed cookies then it is possible for a hacker to create valid cookies for an admin or other user of your application. The recent [SQL injection vulnerability in Rails][1] used knowledge of the secret token to achieve this.

The following post on [keeping your secret token a secret][2] has more on this subject!

## Rails routes.append

Rails Routes is a fundamental part of the Rails framework. It tells Rails what to execute based on the pattern of the incoming request url. Naturally, even the Smallest Rails App needs at least one route! In this case it is the `root` route that is being configured (on the next line actually, line 5). For now lets briefly investigate what exactly `routes.append` does.

`routes.append` works in a very similar way to `routes.draw`. The only apparent difference is that routes.draw does not work before the application is initialized. As far as I can tell `routes.append` (and the `routes.prepend` cousin) allow for [new Rails routes to be added at runtime][3].

In conventional Rails applications routes are &#8216;drawn&#8217; in the routes.rb file by passing a block to the `draw` method call. Within the block, developers have access to a DSL for defining their applications routing. Some of the commonly used methods in the DSL are `root` (as used in the Smallest Rails App) for defining the root url (&#8216;/&#8217;) action within the application. Another very common method call is to `resources` which is a very powerful way to create a RESTful endpoint for a particular resource within your application.

Rails routing is a big topic and for further information it is best to read the [Rails Guide to Routing][4].

## Conclusion

Line 4 of the Smallest Rails App introduces us to a few new concepts namely the `secret_token` and Rails routes. In the next post, I will cover line 5 of the application.

 [1]: https://groups.google.com/forum/?fromgroups=#!topic/rubyonrails-security/DCNTNp_qjFM
 [2]: http://biggestfool.tumblr.com/post/24049554541/reminder-secret-token-rb-is-named-so-for-a-reason
 [3]: http://openhood.com/rails/rails%203/2010/07/20/add-routes-at-runtime-rails-3/
 [4]: http://guides.rubyonrails.org/routing.html

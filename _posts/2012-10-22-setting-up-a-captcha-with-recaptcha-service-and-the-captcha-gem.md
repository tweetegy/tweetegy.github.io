---
title: Setting up a captcha with reCAPTCHA service and the captcha gem
author: jensendarren
layout: post
permalink: /2012/10/setting-up-a-captcha-with-recaptcha-service-and-the-captcha-gem/
image:
  -
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
tags:
  - rails gems captcha recaptcha
---
We all know about captcha. It is a very common way to prevent spam from getting into your inbox, applications, databases, servers etc. It does this by displaying some slightly distorted text such that a machine is unlikely to interpret it correctly but a human will and this is the key. A captcha ensures that it is a real human filling out the form that will keep you busy!

## reCAPTCHA

One of the best services to use for a captcha is [Googles reCAPTCHA service][1]. As usual, it's possible to use this service directly via it's API or use one of the many [plugins or language wrappers][2] available.

Since I am working on a Rails app, I will select one of the Ruby implementations. I opted for the [Recaptcha Gem][3] since the Rack Middleware based gem did not work on the Rails 2.x project I am working on at the moment.

## Setup

Add the following to your gem file:

{% highlight ruby linenos %}
gem "recaptcha", :require => "recaptcha/rails"
{% endhighlight %}

Then [register for a reCAPTCHA API key][4] and add that to your environment config files:

{% highlight ruby linenos %}
#put this in development.rb and in production.rb (separate keys in each so you can test!)
RECAPTCHA_PUBLIC_KEY= 'your-public-key'
RECAPTCHA_PRIVATE_KEY= 'your-private-key'
{% endhighlight %}

The final step for setup is to add the following config file to your initializers directory:

{% highlight ruby linenos %}
#in config/initializers/recaptcha.rb
Recaptcha.configure do |config|
  config.public_key  = RECAPTCHA_PUBLIC_KEY
  config.private_key = RECAPTCHA_PRIVATE_KEY
end
{% endhighlight %}

## View

The Captcha Gem provides helpers for your view to render the actual captcha box. It's as simple as putting the following into your view at the point where you wan the captcha to appear:

{% highlight ruby linenos %}
<%= raw recaptcha_tags %>
{% endhighlight %}

That's it for the view! Of course, this assumes you have put this tag into a form and that you already have error output etc. Moving on now to the server code (in the controller, of course!)

## Controller Code

This is where the verification magic happens. The Captcha Gem provides another helper method that posts to the reCaptcha API server to check if what was submitted is correct. If it is then the method returns true, if not, it will add a custom error message that the captcha is wrong to the model instance. Here is the basic code as you might have it in the create action of your controller:

{% highlight ruby linenos %}#....model setup from params as usual...

@model.valid? #ensures we see all errors on the model in the view if the captcha fails
  if verify_recaptcha(:model => @model, :message => "Please enter the correct captcha!")
    @model.save
    #all ok so do as you would normally
  else
    #model is not valid (that includes the captcha now! probably render the form again?
  end

#...rest of create action
{% endhighlight %}

There are a few things happening here so let me explain. The first line to `@model.valid?` ensures that the models errors array is populated so that if the captcha is incorrect the user will see that message as well as all the model error validation messages. Without that only the captcha message is displayed if not entered correctly. It's nice to show the user all the errors in one go so to prevent server round trips.

The if statement then calls the verify_recaptcha helper which goes off to the Google reCAPTCHA server and validates the captcha has been enetered correctly or not. If not the message is added to the models errors array.

The rest of the controller action is as one would normally write it! So essentially one has replaced:

`if @model.save ....`

with

`if verify_recaptcha(:model => @model, :message => "Please enter the correct captcha!") && @model.save `

pretty neat and straightforward really!

## Conclusion

Very easy to setup, configure and use thanks to the Google service and the well written Captcha Gem!

 [1]: http://www.google.com/recaptcha
 [2]: https://developers.google.com/recaptcha/docs/otherplatforms
 [3]: https://github.com/ambethia/recaptcha/
 [4]: https://www.google.com/recaptcha/admin/create

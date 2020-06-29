---
title: Using Google Analytics API in a Ruby on Rails project
author: jensendarren
layout: post
permalink: /2010/06/using-google-analytics-api-in-a-ruby-on-rails-project/
image:
  -
categories:
  - Ruby Server Programming
tags:
  - analytics
  - api
  - google
  - metrics
  - ruby
---
### Ingredients

*   A new Rails project
*   [The Garb gem][1]
*   A Google Analytics Account </ul>

### Preparation

I assume you know about Rails already. Garb can be installed as follows:

{% highlight bash %}
gem install garb
{% endhighlight %}

Go to your Google Analytics account and fetch the **Account Id** and **Profile Id** that your interested in. You will find your Account Id on your account home page under the link Edit account settings, it is the number after the UA-. You will find your Profile Id, by going back to your account home page, finding the profile in the table, then click the Edit link in that row.the Profile Id is at the top of this page.phew!

If you are not familiar with the Google Analytics API, then you will find the [Data Feed Query Explorer][2] tool useful.

### The Task: Record and display Unique Pageviews from a set list of pages

Now for the fun part! Lets build something that records Unique Page Views from a list of pages in a local database. We need to create a model to store these pages and the unique pageviews. Change into your rails project directory and run the following scaffold script:

{% highlight bash %}
script/generate scaffold page url:string unique_pageviews:integer
{% endhighlight %}

Open up the pages controller that was just generated and add the following method, which sets the class instance variable @profile to an instance of your Google Analytics profile based on the Account Id and Profile Id.

{% highlight ruby linenos %}
def get_analytics_profile
  Garb::Session.login(your_google_analytics_username,your_google_analytics_password)
  accounts = Garb::Account.all
  #Loop all Accounts for account_id
  accounts.each do |account|
    if account.id == your_google_analytics_account_id
      #Loop all Profiles for profile_id
      account.profiles.each do |profile|
       if profile.id == your_google_analytics_profile_id
         @profile = profile
         break
       end
     end
    end
  end
end
{% endhighlight %}

Basically, Garb::Session.login is called passing your username and password to authenticate the session. Next we fetch all accounts and iterate the array until we find the account that matches your Account Id. Then we need to iterate all the profiles within that account until we find the profile that matches your Profile Id.

Now add this method which will update all the unique pageviews in your list of pages:

{% highlight ruby linenos %}
def update_all
  @pages = Page.all
  @pages.each do |page|
      get_analytics_profile if !@profile
      report = Garb::Report.new(@profile)
      report.metrics :unique_pageviews
      report.dimensions :page_path
      report.filters :page_path.contains => page.url
      page.unique_pageviews = report.results.first.unique_pageviews
      page.save
    end
  end
end
{% endhighlight %}

Since you have used a scaffold to create this controller, there should already be an index method present. Add a call to the *update_all* method at the top of the index method so that everytime the index method is called the *update_all* method is called. Here is how your index method should look:

{% highlight ruby linenos %}
def index
  update_all
  @pages = Page.all

  respond_to do |format|
    format.html # index.html.erb
    format.xml  { render : xml => @pages }
  end
end
{% endhighlight %}

### Final Touches

Now all that remains is a little configuration, add some fixtures and we&#8217;re done! You should already have fixtures for your page model. Note that the url does not need to include the domain name, so **/pricing/** or **/sign-up/** are the sorts of values you should put here (obviously ones that exist in your Analytics profile!). The configuration is simply to add:

{% highlight ruby linenos %}config.gem "garb"{% endhighlight %}

in the appropriate place of your environment.rb file.

### Get those metrics!

Now fire up your rails app and navigate to the index for your pages http://localhost:3000/pages/. The call to **update_all** will be made which will update your local database with the metrics and then render the data in the browser in the usual Rails table format.

### Conclusion

Using the [Garb gem][1] makes it extremely easy to work with the Google Analytics API in a Rails project. Obviously this example application can be improved to include caching, error handling (it will bomb out if the Id&#8217;s or urls are not found in your Analytics account) as well as other improvements that you might want to make.

 [1]: http://github.com/vigetlabs/garb/
 [2]: http://code.google.com/apis/analytics/docs/gdata/gdataExplorer.html

---
title: Testing infinite scroll using RSpec and Capybara (without sleep or wait_until)
author: jensendarren
layout: post
permalink: /2013/05/testing-infinite-scroll-using-rspec-without-sleep-or-wait-until/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - capybara
  - javascript
  - rails
  - rspec
  - ruby
---
Recently I added some functionality to an index page that required infinite scrolling. The idea is that on the index view as the user scrolls down the page, and reaches near to the bottom of that page, an Ajax request is made to the server to fetch the next set of records.

In my opinion for this type of view it is much nicer than using pagination (although the solution can still use a pagination Gem in the background, in my case [Kaminari][1]).

## RSpec / Capybara

I will not go into the implementation which is really quite trivial. There is an excellent <a href='http://railscasts.com/episodes/114-endless-page-revised'</a>RailsCast available for &#8216;Endless Page&#8217; solution</a> available already. However, I will demonstrate here how I tested this using RSpec & Capybara. Below is the spec:

{% highlight ruby linenos %}describe "People index" do
  let!(:person) { create(:person) }

  before do
    visit people_path
  end

  it "loads more records when scroll to bottom of page", :js => true do
    default_per_page = Kaminari.config.default_per_page
    create_list(:person, default_per_page)
    Person.count.should > default_per_page
    visit current_path
    page.should have_css('.table-index tbody tr', :count => default_per_page)
    page.execute_script('window.scrollTo(0,100000)')
    page.should have_css('.table-index tbody tr', :count => Person.count)
  end
end
{% endhighlight %}

Firstly notice that we set `default_per_page` to our pagination engine (in this case Kaminari). Then we create that many resources (here its Person resource). The idea is that when Capybara visits the index page (`people_path`) there will be (at least) one resource not loaded on that page yet (until scrolling to the bottom of the page). The important thing is to create *at least* `default_per_page` + 1 resources so that this is testable.

We confirm that there are indeed more resources in the database than the `default_per_page` with the following line:

{% highlight ruby linenos %}Person.count.should > default_per_page
{% endhighlight %}

Once we load the people index page, an assertion is made that the page should contain the same number of resources as defined in the `default_per_page` variable. This is what we expect from our pagination solution anyway.

{% highlight ruby linenos %}page.should have_css('.table-index tbody tr', :count => default_per_page)
{% endhighlight %}

The following line actually scrolls the page down to the bottom which should trigger the Ajax request to load more records.

{% highlight ruby linenos %}page.execute_script('window.scrollTo(0,100000)')
{% endhighlight %}

This is then asserted in the next line that the page now contains `People.count` rows in the table.

{% highlight ruby linenos %}page.should have_css('.table-index tbody tr', :count => Person.count)
{% endhighlight %}

## Sleep(ing) on the job

My first version of this spec was always failing until I put a little `sleep` call before the final assertion. I initially thought this might be necessary to let the DOM properly load following the Ajax call. It turns out, however, that the `sleep` call was required because I was not writing the assert in the correct way. Here is how the last three lines of my spec looked (**don&#8217;t do this!**):

{% highlight ruby linenos %}#rest of spec is the same as above example
page.all('.table-index tbody tr').count.should eq default_per_page
page.execute_script('window.scrollTo(0,100000)') #scroll to the bottom of the page
sleep 1 #NOTE: THIS SHOULD NOT BE NECESSARY.
page.all('.table-index tbody tr').count.should eq Person.count #NOTE: THIS IS NOT THE CORRECT WAY TO ASSERT THIS!
{% endhighlight %}

The problem is in calling `page.all` passing in a selector like this runs immediately and the page is indeed not given a chance to update following the Ajax request. This is why I added a `sleep` call before it. But wait&#8230;I checked on the [Capybara Cheat Sheet][2] and found that there was a `wait_unitl` method so maybe that is what I need!

## Capybara wait_until

So I drop the `wait_until` method into my spec as follows:

{% highlight ruby linenos %}#rest of spec is the same as above example
page.all('.table-index tbody tr').count.should eq default_per_page
page.execute_script('window.scrollTo(0,100000)') #scroll to the bottom of the page
wait_until { page.has_content?(Person.last.name) } #NOTE: THIS DOES NOT WORK IN CAPYBARA 2.x
page.all('.table-index tbody tr').count.should eq Person.count
{% endhighlight %}

However, to my surprise I get the following runtime error when I execute the spec:

{% highlight ruby linenos %}NoMethodError: undefined method 'wait_until' for #&lt;RSpec::Core::ExampleGroup::N...
{% endhighlight %}

After some quick Googling I discovered that [`wait_until` has been removed from Capybara 2.0.0][3] (I am using Capybara 2.1.0, in fact). It was indeed [this blog post][3] that led me to realize I was writing my assertion incorrectly and caused me to refactor it to its final solution:

{% highlight ruby linenos %}page.should have_css('.table-index tbody tr', :count => Person.count)
{% endhighlight %}

This implicitly waits for the content to load (or timesout). For more details on exactly how this works, how to change the timeout etc, I recommend reading the suggested blog post on this subject (or the [Capybara Docs][4]).

## Conclusion

I have to admit that when I wrote the spec initially using the `sleep` method it did smell bad. This is why I went ahead and did a little research into better ways of doing this. If the `wait_until` method was still in Capybara I would have probably (still in error) used that. However, it is a joy to learn that CSS selector methods such as `have_css` implicitly wait until the DOM is loaded making the spec much cleaner!

 [1]: https://github.com/amatsuda/kaminari
 [2]: http://learn.thoughtbot.com/test-driven-rails-resources/capybara.pdf
 [3]: http://www.elabs.se/blog/53-why-wait_until-was-removed-from-capybara
 [4]: http://rdoc.info/github/jnicklas/capybara

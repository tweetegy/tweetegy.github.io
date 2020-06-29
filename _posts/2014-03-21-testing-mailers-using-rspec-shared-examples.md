---
title: Testing Rails ActionMailer using RSpec Shared Examples
author: jensendarren
layout: post
permalink: /2014/03/testing-mailers-using-rspec-shared-examples/
image:
  -
categories:
  - Rails Appplication Programming
tags:
  - actionmailer
  - multipart email
  - rails
  - rspec
  - tdd
  - testing
---
I love ActionMailer in Rails, for many reasons, but one of them is that they are easy to test! I work with mailers a lot in many projects and I do find that I often follow similar testing patterns for each mailer in each project. In this post, I&#8217;ll demonstrate using RSpec shared examples to reuse some of the mailer specs across multiple mailers.

## An ActionMailer

What does it take to create an ActionMailer in Rails 4? A generator, of course!

{% highlight ruby linenos %}
rails g mailer CoolMailer email
{% endhighlight %}

What you should end up with is a basic Mailer class that inherits from `ActionMailer::Base` and related views (+ specs, of course!).

## Testing these puppies!

What we need to do now is actually test these puppies. As we know there are a large number of similarities between most mailers. For example, the `from` property needs to be set to something appropriate for your application or context. The `to` property also needs to be set, of course! We usually want the email to contain a body and we usually want to check that certain content is rendered (or not) within it. Sometimes we want to send multipart emails for supporting both text and html clients.

Other things that may be generic mailer tests:

*   Testing that certain headers are set like &#8216;X-Priority&#8217; and &#8216;X-MSMail-Priority&#8217;
*   Attachments
*   Header content type
*   Subject text
*   Body text<h2Shared Example RSpec for above mailer</h2>

The way I test most of this is using a shared example similar to the one below.

{% highlight ruby linenos %}shared_examples "a well tested mailer" do
  let(:greeting) { "Darren" }
  let(:full_subject) { "#{asserted_subject}:You got mail!" }
  let(:mail) { mailer_class.email(greeting) }

  it "renders the headers" do
    mail.content_type.should start_with('multipart/alternative') #html / text support
  end

  it "sets the correct subject" do
    mail.subject.should eq(full_subject)
  end

  it "includes asserted_body in the body of the email" do
    asserted_body.each do |content|
      mail.body.encoded.should match(content)
    end
  end

  it "should be from 'from@example.com'" do
    mail.from.should include('from@example.com')
  end
end
{% endhighlight %}

Somethings I want to point out about this shared example:

*   Its possible to test multiple content assertions in one go by passing in an array of different content expected. I usually do this for the body.
*   Same goes with the subject. We can test that the subject contains some certain text in the context of our mailer
*   We can swap out different mailers by setting the `mailer_class` variable to a different class (as long, in this case, the template is called &#8216;email&#8217; although this can also be dynamic too if we like).

Notice that we can treat this shared example, as all shared examples can, like a method call where we &#8216;pass in&#8217; certain parameters to work against. You may notice in the above shared examples, variables listed that are not explicitly declared like `full_subject`, `asserted_body` and `mailer_class`. These are set outside of the shared example in the calling spec as shown below:

{% highlight ruby linenos %}describe CoolMailer do
  describe "email" do
    let(:mailer_class) { CoolMailer }
    let(:asserted_subject) { "A cool mail!" }
    let(:asserted_body) { ["This content", "That content"] }

    it_behaves_like "a well tested mailer" do
    end
  end
end
{% endhighlight %}

## A word on multipart html/text emails

You will notice in the shared example a spec for the headers to start with &#8216;multipart/alternative&#8217;. This ensures that our emails are being sent out in both plain text and html. How do we actually get this spec to pass? Fortunately this just works out of the box with ActionMailer! All we need to do is provide **2 templates**; one for html and one for plain text as follows:

{% highlight html %}
#email.html.erb
<h1>CoolMailer#email</h1>
{% endhighlight %}

{% highlight html %}
#email.text.erb
CoolMailer#email
{% endhighlight %}

The main thing that gets this to work is good old Rails &#8216;convention over configuration&#8217;. By simply placing the &#8216;.text.erb&#8217; template and &#8216;.html.erb&#8217; template with the same name (&#8216;email&#8217;) ActionMailer automaticlly sends out a multipart email! Its that easy! Try removing the &#8216;.text.erb&#8217; template and you will see the header spec fail with:

`expected "text/html; charset=UTF-8" to start with "multipart/alternative"`

## Conclusion

The purpose of this post was to demonstrate two things: testing mailers and using shared examples. You don&#8217;t have to use shared examples to test mailers, especially if you only have one mailer in your project. However, most large Rails applications will have multiple mailers so using shared examples makes sense.

As always, an example application is available for download here: [Testing Mailers example application][1]

 [1]: https://github.com/tweetegy/testing_mailers

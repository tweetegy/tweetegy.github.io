---
title: Create reusable UI components and widgets using Backbone.js
author: jensendarren
layout: post
permalink: /2012/03/create-reusable-ui-components-and-widgets-using-backbone-js/
image:
  -
seo_follow:
  - 'false'
categories:
  - JavaScript Client Programming
tags:
  - backbone
  - backbonejs
  - componentes
  - frameworks
  - javascript
  - js
  - ui
  - uicomponents
---
I was building an application in Backbone.js recently and I wanted to experiment with an idea I had to build UI components using Backbone.js. I was given the opportunity when we required a notification element to be displayed on the page. It is possible, to do this in CSS or say using Twitter Bootstrap, however, I wanted to turn this into a basic Backbone.js UI component.

In order to do this we need to think about how the component can be reused in different applications; i.e. not tied directly to any domain specific markup, schema or business logic in the application that is being built. Now, you would probably want to create a separate repository for this project since, after all, it is not strictly meant to live inside an application (unless, of course, your not interested to use the component in any other project other than the one your working on).

## The Template

The template has a single placeholder *message* which can be set in the instance of the model that is used in the application. Note, from the line *class=&#8221;alert fade in&#8221;* that this particular component will work best with [Twitter Bootstrap][1].

{% highlight ruby %}
<script id="notification-template" type="text/template">
  <div>
    <div id="notification" class="alert fade in">
      <a class="close" data-dismiss="alert" href="#">×</a>
      <h3><%= message %></h3>
    </div>
  </div>
</script>
{% endhighlight %}

## The Model

The Notification model has two defaults set: *message* and *status* (which is another model). Notice the namespace *com.tweetegy.ui* used to keep the objects separate from the application.

{% highlight javascript %}
com.tweetegy.ui.NotificationModel = Backbone.Model.extend({
    defaults: {
	message: "Loading...",
	status: com.tweetegy.ui.NotificationStatusMessage
    }
});
{% endhighlight %}

## Notification Status Types

The Notification Status Types Model acts as an enum holding strings to represent the status of each notification that is shown. This is used to control the style of the notification.

{% highlight javascript %}com.tweetegy.ui.NotificationStatusMessage = "NotificatonStatusMessage";
com.tweetegy.ui.NotificationStatusError = "NotificatonStatusError";
{% endhighlight %}

## Backbone View

This is the object that wraps everything up and allows us to reuse this as a component (as we will see later). This is just a standard Backbone View, with an added helper method *updateStyleFromStatus* which changes the style depending on the status of the Notification and *closeNotifcation* that hides it.

The other important thing to note is the line *this.model.bind(&#8220;change&#8221;, this.render);* which is where we use the &#8216;magic&#8217; of Backbone data binding to render our notification anytime the message or status of the applications notification model changes. It means that all we need to do in our application is set either the *message* or the *status* and the view will render itself.

{% highlight javascript %}
com.tweetegy.ui.NotificationView = Backbone.View.extend({
  className: "well form-inline",

  events: {
	 "click .close": "closeNotification"
  },

  initialize: function() {
	 _.bindAll(this, 'render', 'updateStyleFromStatus');
	 this.template = _.template($('#notification-template').html())
	 this.model.bind("change", this.render);
  },

  render: function() {
	 var self = this;
	 $(this.el).html(this.template(this.model.toJSON()));
	 this.updateStyleFromStatus();
	 $(this.el).find("#notification").show();
	 return this;
  },

  closeNotification: function() {
	 $(this.el).find("#notification").hide();
  },

  updateStyleFromStatus: function() {
	 var newClass = "alert fade in";
	 if (this.model.get('status') == window.NotificationStatusError) {
	   newClass = "alert alert-block alert-error fade in"
	 }
	 $(this.el).find("#notification").removeClass().addClass(newClass);
  }
});
{% endhighlight %}

## Using the component

I used it in an application called &#8220;Front Desk&#8221; and initialized it in the bootstrap.js file during application initialziation:

{% highlight javascript %}com.tweetegy.FrontDesk.notification = new com.tweetegy.ui.NotificationModel();
    this._notificationView = new com.tweetegy.ui.NotificationView({
	model: y.f.FrontDesk.notification
    });
    $('div#notification').append(this._notificationView.render().el);
{% endhighlight %}

As mentioned above, in the Backbone View we bind to the models change event to render the view which means all we need to do is change the data in our applications NotificationModel instance (in this case *com.tweetegy.FrontDesk.notification*) and the Notification will automatically be rendered. Here is an example:

{% highlight javascript %}com.tweetegy.FrontDesk.notification.set({message: "Everything loaded! Starting application..."})
{% endhighlight %}

## Conclusion

I have always liked the idea of UI components and using Backbone templates, views and models really makes it easy to build a highly customized UI library for your application, department, company or even to share with the entire community!

I have written another component which uses a Backbone Collection and can be used in a form to collect data from the user. This is a little more complex than the notification example here and I will cover it in a future blog post! Meanwhile, happy coding!

 [1]: http://twitter.github.com/bootstrap/

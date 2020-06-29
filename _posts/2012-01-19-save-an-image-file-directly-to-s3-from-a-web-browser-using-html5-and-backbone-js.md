---
title: Save an image file directly to S3 from a web browser using HTML5 and Backbone.js
author: jensendarren
layout: post
permalink: /2012/01/save-an-image-file-directly-to-s3-from-a-web-browser-using-html5-and-backbone-js/
image:
  -
seo_follow:
  - 'false'
categories:
  - Amazon EC2 S3 Cloud
  - HTML5 Programming
  - JavaScript Client Programming
tags:
  - api
  - File
  - post
  - rest
  - s3
---
This post continues from the [previous post][1] where I show how to [preview a local image using Backbone and HTML5 JavaScript][1]. In this post we will actually upload this file **directly** to an S3 bucket! This post will focus only on that part.

## Overview

The way to avoid any cross-site-scripting restrictions when uploading to S3 is essentially to do two things:

*   Use a form to POST the data to S3
*   Include in that form a Policy and Signature based on the content + your AWS keys

## The Form

Since I am using Backbone.js in this example and since I want to show a preview of the image before uploading it, I realized that I needed to split the form into two templates. The reason was because I needed to be able to refresh the file meta data that is essentially the *key*, *policy* and *signature* required by the AWS S3 API. Here are the two templates:

#### The Image Meta Template

{% highlight html %}
<script type="text/template" id="image-meta-template">
  <input type="hidden" name="key" value='<%= key %>' />
  <input type="hidden" name="acl" value='<%= acl %>' />
  <input type="hidden" name="Content-Type" value='<%= contentType %>' />
  <input type="hidden" name="AWSAccessKeyId" value='<%= AWSAccessKeyId %>' />
  <input type="hidden" name="success_action_redirect" value='<%= successActionRedirect %>' />
  <input type="hidden" name="x-amz-meta-filename" value='<%=filename %>' />
  <input type="hidden" name="Policy" value='<%= POLICY %>' />
  <input type="hidden" name="Signature" value='<%= SIGNATURE %>' />
</script>
{% endhighlight %}

#### The Image File Template

{% highlight erb %}
<script type="text/template" id="image-file-template">
  <form id="formBlob" action="<%= bucket %>.s3.amazonaws.com" method="post" enctype="multipart/form-data">
    <input id="myImage" type="file" name="file" />
    <input id="btnSave" type="submit" value="Save Image" />
  </form>
</script>
{% endhighlight %}

The data is passed to these templates from the Backbone View. However, when we select a new file we need to first update the preview (this template is discussed in the [previous post][1]) and then update the image meta template only (not the image file template because that is already correct). So this is the reason I have two templates here.

The interesting attributes of the image meta template are: *key*, *POLICY* and *SIGNATURE* which are all required by the AWS S3 API. The key is the simplest of the three, so let&#8217;s start with that.

## The Key

The key is simply the bucket key we need to use in order to save (and later retrieve) the image back. In this example, I have the key made up of two attributes on the Backbone Model: folder and filename. The folder is set during the creation of the model in the bootstrap.js file and the filename is set when the user selects a file using the *myImage* file input. Right after the file has been sucessfully loaded into the preview, I call a function &#8220;updatePoilcy&#8221; which first sets the key attribute on the Model like so:

{% highlight javascript %}updatePolicy: function(){
	var key = this.get('folder') + this.get('filename');
	this.set({key: key});

	//more code later...
}
{% endhighlight %}

## The Policy

The Policy is a JSON document that is then converted to Base64 encoding and set as a property on the Model. The code to do this is also in the updatePolicy function (hence the name!). Note that we are getting the data directly from the Model to build this Policy and that it must (and **will**, thanks to Backbone binding!) match with the values in the form template. Here is that code:

{% highlight javascript %}updatePolicy: function(){
//code to set the key attribute on the model

POLICY_JSON = { "expiration": "2012-12-01T12:00:00.000Z",
    "conditions": [
	["eq", "$bucket", this.get('bucket')],
	["starts-with", "$key", this.get('key')],
	{"acl": this.get('acl')},
	{"success_action_redirect": this.get('successActionRedirect')},
	{"x-amz-meta-filename": this.get('filename')},
	["starts-with", "$Content-Type", this.get('contentType')]
    ]
  };

var policyBase64 = Base64.encode(JSON.stringify(POLICY_JSON));
//Set the Policy on the Model so that it is then sent with the Form payload
this.set({POLICY: policyBase64 });

//more code later...
}
{% endhighlight %}

## The Signature

The final part of this puzzle is the Signature. This is generated using the Policy and your AWS Secret key and is encrypted using SHA1 like so:

{% highlight javascript %}updatePolicy: function(){
//previous code (shown above)
var secret = this.get('AWSSecretKeyId');
var policyBase64 = Base64.encode(JSON.stringify(POLICY_JSON));
var signature = b64_hmac_sha1(secret, policyBase64);

this.set({SIGNATURE: signature });
}
{% endhighlight %}

The full *updatePolicy* function looks like this. Note that this function is called on the Model initialize and on any data changes (i.e. when the user selects a new image).

{% highlight javascript %}updatePolicy: function(){
	    var key = this.get('folder') + this.get('filename');
	    this.set({key: key});

	    POLICY_JSON = { "expiration": "2012-12-01T12:00:00.000Z",
			    "conditions": [
				["eq", "$bucket", this.get('bucket')],
				["starts-with", "$key", this.get('key')],
				{"acl": this.get('acl')},
				{"success_action_redirect": this.get('successActionRedirect')},
				{"x-amz-meta-filename": this.get('filename')},
				["starts-with", "$Content-Type", this.get('contentType')]
			    ]
			  };

	    var secret = this.get('AWSSecretKeyId');
	    var policyBase64 = Base64.encode(JSON.stringify(POLICY_JSON));
	    var signature = b64_hmac_sha1(secret, policyBase64);

	    this.set({POLICY: policyBase64 });
	    this.set({SIGNATURE: signature });
	}
{% endhighlight %}

## Success Action Redirect

The *successActionRedirect* attribute on the model is worth a mention. In this example, I set it to the same page that sent the POST request. If you only want to send data to S3 this is fine, but what if you want to sent some of the meta data to another separate API? There are several ways to do this but one way would be to set this attribute to a service that sends the bucket and key to a backend queue. That queue can be periodically &#8216;popped&#8217; via a cron job that makes a GET request to S3 to retrieve all the meta data saved there for that image file. This data can then be saved to a separate API at this point. This gives the developer full control over where the file meta data is transformed and stored.

Another thing to note is the attribute *x-amz-meta-filename*. This is actually a custom attribute and the developer can set any number of these with the POST request. They must start with &#8220;x-amz-meta-&#8221; but that&#8217;s about it! So you could have &#8220;x-amz-meta-width&#8221; & &#8220;x-amz-meta-height&#8221; which could be calulated using a HTML5 Canvas set and set as properties on the Model. As long as the Policy and the Form contain these same attributes, they will all be saved along with the image when the form is POST to S3. Very handy!

## Conclusion

This is a basic example to show how to upload a file directly to S3 from the browser. There are still plenty of unanswered questions such as being able to upload multiple files or upload a file using AJAX and then post some meta data to a separate API (all directly from the browser). These issues can be the subjects of future posts.

## Code

The code for this example is available [here][2]

To run the example you need to serve the folder from a HTTP server. I usually use Python SimpleHTTPServer for basic examples like this one. Thus, by default, if you CD into the example directory and run **python -m SimpleHTTPServer** then the application will be available at <http://localhost:8000>. You also need to create an S3 account and update **YOUR-BUCKET**, **YOUR-ACCESS-KEY** and **YOUR-SECRET-KEY** in bootstrap.js. Oh! and it will only work in the latest WebKit based browsers. I tested it in Chrome 16.0.912.63 beta. Good luck!

## Tutorials

[POST Example from AWS][3]
[Upload S3 files directly with AJAX][4]
[Another example from AWS][5]

 [1]: /2012/01/preview-a-local-image-using-backbone-and-html5-javascript/
 [2]: https://github.com/tweetegy/save_image_directly_to_s3_using_backbone
 [3]: http://s3.amazonaws.com/doc/s3-example-code/post/post_sample.html
 [4]: http://blog.danguer.com/2011/10/25/upload-s3-files-directly-with-ajax/
 [5]: http://docs.amazonwebservices.com/AmazonS3/latest/dev/HTTPPOSTExamples.html

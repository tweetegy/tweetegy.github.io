---
title: Basic multiple image gallery upload HTML5 and Backbone application
author: jensendarren
layout: post
permalink: /2012/01/basic-multiple-image-gallery-upload-html5-and-backbone-application/
image:
  -
seo_follow:
  - 'false'
categories:
  - HTML5 Programming
  - JavaScript Client Programming
  - Ruby Server Programming
tags:
  - backbone
  - html5
  - javascript
  - ruby
  - sinatra
---
This is really a very basic example, with the main aim to get anyone started with a simple image upload and persist application using HTML5 and Backbone. This post is essentially a continuation of a previous post where I explained how to [preview a local image using HTML5 and Backbone][1]. The main expansion here is that I am adding the capability to:

*   Preview multiple images using a multiple file input
*   Add all the images to a Backbone collection and persist that collection to a server

## Backbone Templates

I'll start with the templates again. In this application I have three templates: one for selecting multiple images, one for displaying a single image preview and a container template for the header text and a placeholder for the collection.

The select multiple images template looks as follows:

{% highlight html %}
<script type="text/template" id="gallery-selection-template">
  <input id="myGallery" type="file" name="file" multiple /><br />
  <button id="saveGallery">Save Gallery</button>
</script>
{% endhighlight %}

The single image preview template is as follows:

{% highlight html %}
<script type="text/template" id="image-template">
  <img class="thumb" src='<%= data %>' title='<%= filename %>' />
</script>
{% endhighlight %}

Finally, the container is the most basic template, since it's just a title and a placeholder for the collection:

{% highlight html %}
<script type="text/template" id="gallery-template">
  <h2>Image Gallery</h2>
  <output id="thumbnails" />
</script>
{% endhighlight %}

There is nothing really particullarly special to mention with these templates, so let's move on.

## Backbone Views

I only want to focus on the enhancements I have made which is being able to preview a collection of images and save them to the server via a Backbone collection. Therefore, I will focus on one view here: *GalleryView*. Actually, this is a standard way of building a view that renders a collection of sub-views in it's render method. Here I get a handle on the template *gallery-template* and append the rendered out *ImageView* for each Image in the Backbone collection for the Image Gallery:

{% highlight javascript linenos %}
window.GalleryView = Backbone.View.extend({
template: _.template($("#gallery-template").html()),

initialize: function() {
  _.bindAll(this, 'render');
  this.collection.bind('add', this.render);
},

render: function(){
  var $images,
  collection = this.collection;

  $(this.el).html(this.template({}));
  $images = this.$("#thumbnails");
  this.collection.each(function(image) {
      var view = new ImageView({
          model: image,
          collection: collection
      });
      $images.append(view.render().el);
  });

  return this;
}
});
{% endhighlight %}

## Backbone Collections

The Gallery collection is responsible for loading the image data into Backbone Image models and adding them to the collection. There is a function *setFromFiles* which expects a FileList object passed to it from the multiple file input. Here is the collection code:

{% highlight javascript %}window.Gallery = Backbone.Collection.extend({
	model: Image,
	url: "/images",

	setFromFiles: function(files) {
	    this.reset();
	    self = this;

	    for (var i = 0, f; f = files[i]; i++) {
		var reader = new FileReader();

		reader.onload = (function(theFile, theId) {
		    return function(e) {
			image = new window.Image();
			image.set({id: theId})
			image.set({filename: theFile.name});
			image.set({data: e.target.result});
			self.add(image);
		    };
		})(f);

		reader.readAsDataURL(f,i);
	    }
	}
});
{% endhighlight %}

The code that calls the *setFromFiles* method is actually in the *ImageSelectionView* (that renders the *gallery-selection-template*) and is called when a change event occurs on the multiple file input. The function looks like this:

{% highlight javascript %}dispatchUpdateGalleryPreview: function(e) {
	this.collection.setFromFiles(e.target.files);
}
{% endhighlight %}

The *e.target.files *is the *FileList* object that the *setFromFiles* function expects. Note I have prepended the function name with *dispatch* because it&#8217;s good practice in production code to have the application event driven and therefore this function would actually dispatch a custom event instead of calling the function directly on the collection.

## Backbone Sync

Finally, we arrive at the persistence part! On the client side we use a little more Backbone. In this case, I simply want to save the entire collection, so I will use *Backbone.Sync* as follows:

{% highlight javascript %}Backbone.sync("create", this.collection);
{% endhighlight %}

This will cause Backbone to perform a POST request to the server to the location: &#8216;/images&#8217; which is basically the url set in the Gallery collection earlier. One the server a Ruby file is waiting to process the response. In this example, I am using Sinatra server. Of course, developers are free to use any server technology they want! For completeness, here is the server side Ruby code:

{% highlight ruby linenos %}post "/images" do
  payload = JSON.parse(request.body.read.to_s)

  payload.each do |image|
    data = image["data"]
    i = data.index('base64') + 7
    filedata = data.slice(i, data.length)

    File.open(image["filename"], 'wb') do |f|
      f.write(Base64.decode64(filedata))
    end
  end

end
{% endhighlight %}

## Conclusion

This is a very basic application which demonstrates selecting, previewing and saving multiple image files to a Ruby Sinatra endpoint using Backbone and HTML5 on the client. There is no functionality to manage a collection of images, for example, edit or delete. However, I feel this example is enough to get started to build a fuller Image Gallery application! Many features could be implemented in this application including using the HTML5 Canvas to automatically resize images or allow the user to alter the images directly before saving. On the server side, the Ruby script could actually push the images to S3 instead of saving to the file system, for example.

## Code

The code for this example is [here][2].

## Tutorials

[Great tutorial using FileReader by HTML5 Rocks][3]

 [1]: /2012/01/preview-a-local-image-using-backbone-and-html5-javascript/
 [2]: https://github.com/tweetegy/multiple_image_gallery_upload_HTML5_and_Backbone_application
 [3]: http://www.html5rocks.com/en/tutorials/file/dndfiles/

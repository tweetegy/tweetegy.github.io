---
title: Update ZoneEdit NameServers via a Restful API in Ruby
author: jensendarren
layout: post
permalink: /2009/07/update-zoneedit-nameservers-via-a-restful-api-in-ruby/
image:
  -
seo_follow:
  - 'false'
categories:
  - Programming
  - Ruby Server Programming
tags:
  - rest
  - ruby
  - zoneedit
---
I recently had to programmatically alter entries in [ZoneEdit][1]. While there is some documentation, it is difficult to get started quickly. This is especially true if you want to use Ruby since there are no examples written in Ruby that I could find. So I wrote a simple class to do just what I need and here it is:

{% highlight ruby linenos %}
require 'net/http'
require 'net/https'
require 'uri'

class ZoneEdit

	#Initialize with the ZoneEdit username and password credentials
	def initialize(username, password)
		@@username = username
		@@password = password
	end

	#This method can be used to change a dns record in ZoneEdit
	#Pass in the account username and zone you want to manage as well as the dnsfrom (sub-domain) forward address and 0 or 1 to shadow (cloak) or not
	def web_forward(user,zone,dnsfrom,forward,shadow)
		return send_command("command=ChangeRecord&user=#{user}&zone=#{zone}&type=WF&dnsfrom=#{dnsfrom}&forward=#{forward}&shadow=#{shadow}")
	end

	private
	def send_command(command)
		puts "https://www.zoneedit.com/auth/admin/command.html?#{command}"
		@http=Net::HTTP.new('www.zoneedit.com', 443)
		@http.use_ssl = true
		@http.start() {|http|
			req = Net::HTTP::Get.new("/auth/admin/command.html?#{command}")
			req.basic_auth @@username, @@password
			response = http.request(req)
			print response.body
		}
	end

end
{% endhighlight %}

It's usage is simply as follows:

{% highlight ruby linenos %}
ze = ZoneEdit.new("someuser", "apassword")
ze.web_forward "someuser", "somedomain.com", "somesubdomain", "http://anewdomain.com", "1"
{% endhighlight %}

 [1]: http://www.zoneedit.com/

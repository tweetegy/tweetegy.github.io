---
title: Compare Amazon S3 to EBS data read performance
author: jensendarren
layout: post
permalink: /2009/12/compare-amazon-s3-to-ebs-data-read-performance/
image:
  -
seo_follow:
  - 'false'
categories:
  - Amazon EC2 S3 Cloud
  - Bash Scriping
tags:
  - amazon
  - ebs
  - performance
  - ruby ec2
  - s3
  - test
---
If you are going to store data in the cloud that requires frequent reading (such as a database file) then you should consider using EBS. Just to reinforce that statement, lets compare the read performance between S3 and EBS.

I used the following straightforward Ruby program to perform the test. To try this yourself, just fill in the access keys and configure the local folder, bucket and files with your own setup:

{% highlight ruby linenos %}require 'benchmark'
require 'rubygems'
require 'aws/s3'

AWS::S3::Base.establish_connection!(:access_key_id => YOUR_ACCESS_KEY_ID, :secret_access_key => YOUR_SECRET_ACCESS_KEY)

Benchmark.bm do |x|
  x.report("disk (1 read): ") { 1.times {File.read('/your-ebs-mount/1.xml')}}
  x.report("s3 (1 read): ") { 1.times { AWS::S3::S3Object.value '1.xml', "your-s3-bucket" } }
  x.report("disk (1000 read): ") { 1000.times {File.read('/your-ebs-mount/1.xml')}}
  x.report("s3 (1000 read): ") { 1000.times { AWS::S3::S3Object.value '1.xml', "your-s3-bucket" } }
end
{% endhighlight %}

This code was executed in a small EC2 instance on the cloud and the output is shown below. As you can see, disk read is not only faster (as expected) but very much faster at about 2000x!!

Of course, S3 is best used for backing up files, packing up a EC2 image and as a source to Amazon&#8217;s CDN via CloudFront, which explains the massive speed difference here. The purpose of this test was to simply make a side-by-side comparison of reading the same file from S3 compared with EBS.

{% highlight bash %}user     system      total        real
disk (1 read):   0.000000   0.000000   0.000000 (  0.000126)
s3 (1 read):   0.010000   0.000000   0.010000 (  0.308293)
disk (1000 read):   0.030000   0.020000   0.050000 (  0.116382)
s3 (1000 read):   0.710000   0.100000   0.810000 (259.014794)
{% endhighlight %}

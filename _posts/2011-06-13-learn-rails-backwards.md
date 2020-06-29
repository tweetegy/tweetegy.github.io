---
title: Learn Rails Backwards
author: jensendarren
layout: post
permalink: /2011/06/learn-rails-backwards/
image:
  - 
seo_follow:
  - 'false'
categories:
  - Rails Appplication Programming
tags:
  - course
  - idea
  - learning
  - rails
  - study
  - teaching
  - technique
---
I sometimes run free Rails courses locally here. One of the things that I have learnt is that it seems to be best to teach Rails &#8216;backwards&#8217;. What do I mean by that? Well it depends on how you define &#8216;forwards&#8217;! If you practice TDD then a high level process would be to:

1.  Write your (failing) test in RSpec
2.  Write Rails code to pass the test

This means knowing Ruby before you even start step one above, of course. However, when teaching Rails, I like to do things in reverse, namely:

1.  Learn how to write Rails code
2.  Learn how to write RSpec code
3.  Learn how to write Ruby code

Now, of course it&#8217;s *all* Ruby, but the point is that the order when teaching / learning Rails is reversed compared with what is done in practice. Of course, this approach does have its good and bad points.

## Good Points

*   It&#8217;s easier to jump straight into a framework with some simple examples and exercises to wow the students
*   A lot of the time, Students can start building something cool right after the first lesson
*   It&#8217;s especially good for new programmers as their interest in what is going on behind the scenes starts to seep through subconsciously.

## Bad Points

*   It may be the beginning of a lifetime of bad habits (a.k.a &#8220;where are your tests!&#8221;).
*   Following on from the above point, tests make for beautifully simple code &#8211; only do what you need and no more. If the student gets into the habit of writing tests after then the code will not be so simple.
*   For some students, maybe the more experienced ones, it might be better off with the &#8216;forward&#8217; approach: Learn Ruby, TDD / RSpec, Rails.

What are your thoughts on this?
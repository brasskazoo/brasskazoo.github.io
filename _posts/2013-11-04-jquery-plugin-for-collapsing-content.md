---
layout: post
title: jQuery plugin for collapsing content - readmore.js
date: '2013-11-04T16:27:00.001+11:00'
categories: web-development jquery

author: brasskazoo
tags:
- Javascript
- jQuery
modified_time: '2013-11-04T16:27:39.634+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-6789895976805388326
blogger_orig_url: http://blog.brasskazoo.com/2013/11/jquery-plugin-for-collapsing-content.html
---

If you're looking for a simple method to collapse and expand blocks of HTML 
content, I highly recommend the jQuery plugin 
[readmore.js](http://jedfoster.com/Readmore.js/) by Jed Foster. 

Highly customisable, and implemented with a single function call against any 
id, class or element, e.g.: 

{% highlight javascript %}
$('p').readmore();
{% endhighlight %}

All CSS, more/less links and animation is taken care of automatically! 

One thing to pay attention to however, is the height of the collapsed text, as 
a mis-alignment can cause text lines to be chopped in half horizontally. This 
is fixed by tweaking the `maxHeight` parameter, or alternatively you might be 
able to do something fancy with CSS to pretty it up. 

Another tweak to consider is replacing the default 'read more' text with an 
image or button. 

Project page: 
[http://jedfoster.com/Readmore.js](http://jedfoster.com/Readmore.js)

Github: 
[https://github.com/jedfoster/Readmore.js](https://github.com/jedfoster/Readmore.js) 
---
layout: post
title: Loading external javascript into Blogger
date: '2012-01-04T23:13:00.000+11:00'
categories:

author: brasskazoo
tags:
- Blogger
- blogging
- Javascript
modified_time: '2012-01-04T23:14:41.637+11:00'
thumbnail: http://1.bp.blogspot.com/-VL5fSUe-ORA/TwQ7QQNz-LI/AAAAAAAAAC0/hFISWo7ra30/s72-c/blogger-js-postbody.png
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-8884758075609902888
blogger_orig_url: http://blog.brasskazoo.com/2012/01/loading-external-javascript-into.html
---

Unfortunately, Blogger doesn't make it particularly straight-forward to 
include 3rd-party scripts in my blog. 
Things like Alex Gorbatchev's [syntax 
highlighter](http://alexgorbatchev.com/SyntaxHighlighter/) or other 
externally-hosted scripts (e.g. Google code, github etc.) I may want to be 
available in all my posts. 
There are a few options to get javascript into a post:<a name='more'></a> 
1. Include in the post body 
1. In a template gadget 
1. In the template HTML 
But keep in mind that Javascript best 
practices<sup>[1](http://www.blogger.com/blogger.g?blogID=4222683507658340156#ftn.id394062)</sup> 
suggest placing script tags as close to the end of the body tag as possible. 
Note these options also apply if you're including your own arbitrary 
javascript in your posts. 
##    Post bodyAs simple as chucking the `&lt;script&gt;` tag in the HTML view 
of the post. Might be good for a one-off/single-use script, but I've found 
that it can sometimes affect the white space of the post. 
Also if the script is used regularly it would be a pain doing this for each 
post. 
<table align="center" cellpadding="0" cellspacing="0" 
class="tr-caption-container" style="margin-left: auto; margin-right: auto; 
text-align: center;"><td style="text-align: center;">[<img border="0" 
height="66" 
src="http://1.bp.blogspot.com/-VL5fSUe-ORA/TwQ7QQNz-LI/AAAAAAAAAC0/hFISWo7ra30/s400/blogger-js-postbody.png" 
width="400" 
/>](http://1.bp.blogspot.com/-VL5fSUe-ORA/TwQ7QQNz-LI/AAAAAAAAAC0/hFISWo7ra30/s1600/blogger-js-postbody.png)<td 
class="tr-caption" style="text-align: center;">Javascript in the post body 
##    Template GadgetCreating an HTML/Javascript gadget on the template allows 
arbitrary code to be chucked into the layout (thus available for all posts). 
But this again can affect the whitespace of the page, and leave a blank panel 
where you may not want it! 
<table align="center" cellpadding="0" cellspacing="0" 
class="tr-caption-container" style="margin-left: auto; margin-right: auto; 
text-align: center;"><td style="text-align: center;">[<img border="0" 
height="200" 
src="http://1.bp.blogspot.com/-nZLYlXBlUH8/TwQ2J-VOdPI/AAAAAAAAACU/YQyk12Gpm4Q/s200/blogger-html-widget.png" 
width="190" 
/>](http://1.bp.blogspot.com/-nZLYlXBlUH8/TwQ2J-VOdPI/AAAAAAAAACU/YQyk12Gpm4Q/s1600/blogger-html-widget.png)<td 
class="tr-caption" style="text-align: center;">HTML/Javascript widget 
##    Template HTMLBy editing the HTML for the template, the `&lt;script&gt;` 
tags can be placed right on top of the `&lt;/body&gt;` tag. 
Edit the template by going to Template -&gt; Edit HTML. Read the warning and 
click 'Proceed'. 
Searching for the string `"text/javascript"` should point out an existing 
block of javascript that is directly before the `&lt;/body&gt;` tag, so it is 
just a matter of including additional `&lt;script&gt;` tags in that area. 
<table align="center" cellpadding="0" cellspacing="0" 
class="tr-caption-container" style="margin-left: auto; margin-right: auto; 
text-align: center;"><td style="text-align: center;">[<img border="0" 
height="291" 
src="http://3.bp.blogspot.com/-F7J-Sr55Qrc/TwQ2ft7fAjI/AAAAAAAAACo/i8eBdpB0rXs/s400/blogger-template-html.png" 
width="400" 
/>](http://3.bp.blogspot.com/-F7J-Sr55Qrc/TwQ2ft7fAjI/AAAAAAAAACo/i8eBdpB0rXs/s1600/blogger-template-html.png)<td 
class="tr-caption" style="text-align: center;">Editing the template HTMLThis 
is clearly the best solution from the best-practice point of view, but be 
warned: if the blog template is changed the modifications to the template HTML 
will be lost! 


<sup>[[1](http://www.blogger.com/blogger.g?blogID=4222683507658340156#id394062)]</sup>By 
placing the script tags at the end of the body section, it does not block the 
loading of the main content and means the javascript can be loaded and applied 
unobtrusively. See related notes on 
[developer.yahoo.com](http://developer.yahoo.com/performance/rules.html#js_bottom) 
for more info. 
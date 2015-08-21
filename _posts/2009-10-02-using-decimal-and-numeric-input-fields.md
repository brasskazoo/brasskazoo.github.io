---
layout: post
title: Using decimal and numeric input fields in Android
date: '2009-10-02T07:18:00.000+10:00'
categories: software-development android

author: brasskazoo
tags:
- Java
- UI
- Android
- numeric
modified_time: '2012-01-02T19:32:03.952+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-5657050917044900217
blogger_orig_url: http://blog.brasskazoo.com/2009/10/using-decimal-and-numeric-input-fields.html
---

Say I have a measurement field in my app that takes a decimal: 
<img class="alignnone size-full wp-image-15" title="android-decimal" 
src="http://codingbone.wordpress.com/files/2009/10/android-decimal.png" 
alt="android-decimal" width="375" height="76" /> 

Google's dev guide says that [floats are
bad](http://developer.android.com/guide/practices/design/performance.html#avoidfloat), 
because the embedded hardware does not have floating point number support like 
desktop CPUs do.

And [creating objects should be 
avoided](http://developer.android.com/guide/practices/design/performance.html#object_creation) 
too, if possible, because it is an expensive operation in an environment with 
limited resources. Garbage collection of objects used in UI code can lead to 
'hiccups' in the user experience too. 

So if a float is out, that leaves us with double, right? The best solution
appears to be this:

{% highlight java %}
double value = Double.parseDouble(txtInput);
{% endhighlight %}

I think _parseDouble_ is better than _valueOf_, since we
probably don't want to be doing any auto-unboxing. 

*Note*: I include these lines in the layout xml to restrict the field to a
single-line decimal input: 

{% highlight xml %}
<EditText
    .... 
    android:singleLine="true" 
    android:inputType="numberDecimal" 
    ... 
>
{% endhighlight %}

The _inputType_ also sets the soft keyboard (popup keyboard) to the
numeric keyboard. 
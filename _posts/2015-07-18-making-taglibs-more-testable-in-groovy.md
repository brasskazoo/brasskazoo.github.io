---
layout: post
title: 'Making taglibs more testable in Groovy & Grails - a better way to construct using Groovy Builders '
date: '2015-07-18T16:44:00.000+10:00'
categories: software-development code-quality grails

author: brasskazoo
tags:
- taglib
- unit testing
- html forms
- Web Applications
- Software Development
- Code Quality
- Groovy
- Grails
- HTML5
modified_time: '2015-07-18T16:44:06.980+10:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-12385599949653821
blogger_orig_url: http://blog.brasskazoo.com/2015/07/making-taglibs-more-testable-in-groovy.html
---

Writing custom [taglibs](http://grails.github.io/grails-doc/2.2.1/ref/Tag%20Libraries/Usage.html) 
can easily become messy - explicit HTML, appending ad-hoc strings to a
`StringBuilder`, adding attributes based on tag arguments can easily turn 
relatively simple code into an unmaintainable mess. Not to mention trying to 
trace logic paths to track down bugs.. 

## Using `MarkupBuilder`

One strategy I have used to simplify construction of HTML in taglibs is to use 
Groovy's [`MarkupBuilder`](http://docs.groovy-lang.org/latest/html/api/groovy/xml/MarkupBuilder.html) 
to create content is a cleaner way than raw Strings. 

`MarkupBuilder` extends [`BuilderSupport`](http://docs.groovy-lang.org/latest/html/api/groovy/util/BuilderSupport.html), 
which is a base class to provide support to arbitrary nested objects, allowing 
the ability to create DSL-like trees. 

In this case `MarkupBuilder` can help to create structured, nested additions 
to an object tree that when rendered to a `String`, outputs valid HTML to be 
used in place of the tag. 

An example (taken from the class documentation): 

{% highlight java %}
new MarkupBuilder().root {
    a( a1:"one" ) {
      b { mkp.yield("ABC") }
    c( a2:"two", "blah" )
  }
}
{% endhighlight %}

When rendered to a `StringWriter`, will output: 

{% highlight html %}
<root>
   <a a1="one">
     <b>ABC</b>
     <c a2="two">blah</c>
   </a>
</root>
{% endhighlight %}

Notice that there are tags (`root`, `a`, `b`, `c`), attributes (`a1`, `a2`), and text content included in the objects added to the builder. 

If we take the following code: 
{% highlight java %}
StringBuilder stringBuilder = new StringBuilder()
stringBuilder << "<div class='${buttonClass}'"
stringBuilder << " data-toggle='buttons' id='${parentId}'>"
stringBuilder << "Div Text Content"
stringBuilder << "</div>"
{% endhighlight %}

The `MarkupBuilder` equivalent could be:

{% highlight java %}
def sw = new StringWriter()
def builder = new MarkupBuilder(sw)

builder.omitNullAttributes = true
builder.div(class: buttonClass, data-toggle: "buttons", id: parentId) {
    mkp.yield("Div Content")
}

out << sw.toString()
{% endhighlight %}

I reckon this looks _much_ cleaner, simplier and maintainable.

You can pass in whatever attributes you want to the parameters, and a node 
will be attached to the tree with the element name and attributes. The 
`omitNullAttributes` setting I find useful for when an attribute might be 
missing (e.g. a value being passed to an input field) without disturbing the 
code. 

__FYI__: mkp is a reference to the [MarkupBuilderHelper](http://docs.groovy-lang.org/latest/html/api/groovy/xml/MarkupBuilderHelper.html) class. 

Another example, with nested elements using Twitter Bootstrap CSS classes: 
{% highlight java %}
def sw = new StringWriter()
def builder = new MarkupBuilder(sw)

builder.omitNullAttributes = true
builder.div(class: "input-group") {
    label(for: id, class: "input-group-addon") {
        mkp.yield("Username")
    }
    span(class: "glyphicon glyphicon-user")
    input(required: required ? "" : null, placeholder: placeholder, type:
"text", name: name, id: id, class: classString, value: value)
}
{% endhighlight %}

Many of the attributes here are variables passed to the tag function. 

## Testability

So how does using MarkupBuilder help make the code more testable?

Probably the most pain from maintaining taglib tests comes from inconsistencies in formatting - namely: 

* Whitespace, indents - makes maintaining text matching assertions tedious 
* Use of different quote marks around attributes - while syntactically correct, more tedium, and potentially causing rendering issues when mixed with grails `${var}` placeholders 
* Validating HTML At the very least, using MarkupBuilder makes the generated HTML more reliable and consistent, meaning that tests can be more resilient. It is difficult enough when some tests must be reduced to textual comparisons. 

By the way, the official grails documentation [here](https://grails.github.io/grails-doc/latest/guide/testing.html#unitTestingTagLibraries) outlines the basic ways of testing output using Spock Specification framework. 
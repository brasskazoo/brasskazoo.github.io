---
layout: post
title: Grails 2.5.0 - Disabling autoTimestamp during Integration Tests
date: '2015-06-11T17:25:00.000+10:00'
categories: software-development grails

author: brasskazoo
tags:
- integration testing
- Groovy
- Grails
modified_time: '2015-06-11T17:25:00.201+10:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-917739154039294603
blogger_orig_url: http://blog.brasskazoo.com/2015/06/grails-250-disabling-autotimestamp.html
---

At some point between Grails 2.2 and 2.5, the functionality around the 
[`autoTimestamp`](http://grails.github.io/grails-doc/2.5.0/ref/Database%20Mapping/autoTimestamp.html) 
flag has changed in a way that has caused it to not function during 
integration tests. 
What this means is that if you are testing integrations that involve the 
creation of a domain that has the `autoTimestamp` flag, you may be hit with 
not-nullable errors: 

{% highlight java %}
org.springframework.dao.DataIntegrityViolationException: not-null property references a null or transient value : com.your.package.YourDomain.dateCreated; nested exception is org.hibernate.PropertyValueException: not-null property references a null or transient value : com.your.package.YourDomain.dateCreated
{% endhighlight %}

Obviously the reason for this is that the `dateCreated` (and `lastUpdated` if 
you use it) field is not being set in the way that it would if the application 
was running.

If you are unable to manually add these in your test, (e.g. in my case the 
object was being created and saved by a service that was under test - no 
chance to mock or modify before the save), you might look to somehow disabling 
the `autoTimeout` and setting the default value 
An answer lies in `Bootstrap.groovy`, where we can catch the grails context 
and modify the domain class before the tests are executed. 

Note that if you're developing a plugin, you wouldn't normally have a
`Bootstrap.groovy` file, since its unlikely you're interacting with a 
database. But you can create it manually, and it won't get packaged up in your 
plugin build. 
<h2 id="bootstrap">Bootstrap</h2>In your `Bootstrap.groovy`, you'll need the 
following function. This will disable the `autoTimestamp`, and set the 
`dateCreated` and `lastUpdated` fields to the current date: 

{% highlight groovy %}

    def disableAutoTimestamp(Class domClass) {
    def grailsSave = domClass.metaClass.pickMethod('save',
        [Map] as Class[])

    domClass.metaClass.save = { Map params ->

        def m = new GrailsDomainBinder()
            .getMapping(delegate.getDomainClass())
        println("Disabling autoTimestamp for ${domClass.name}") 

        if (m?.autoTimestamp) m.autoTimestamp = false 

        def colList = [] 
        domClass.declaredFields.each { 
            if (!it.synthetic) 
                colList.add(it.name.toString()) 
        } 
        if (colList.contains("dateCreated")) { 
            println "Setting ${domClass.simpleName}.dateCreated to now" 
            domClass.metaClass.setProperty(delegate, "dateCreated",
                new Date(System.currentTimeMillis()))
                // Update this when using Java8
        } 

        if (colList.contains("lastUpdated")) { 
            println "Setting ${domClass.simpleName}.lastUpdated to now" 
            domClass.metaClass.setProperty(delegate, "lastUpdated",
                new Date(System.currentTimeMillis()))
        } 
        grailsSave.invoke(delegate, [params] as Object[]) 
    } 
} 
{% endhighlight %}

_Note that prior to grails 2.4, the `GrailsDomainBinder().getMapping()` was static._

The [stackoverflow 
question](http://stackoverflow.com/questions/28735133/pull-domain-mapping-in-bootstrap-and-modify-it-grails) 
that I based this code on iterated through all classes with: 

{% highlight java %}
grailsApplication.domainClasses.each { gdc ->
    def domClass = gdc.clazz 
    ... 
} 
{% endhighlight %}

I found that this didn't work when proccessing all domains, so I specify them 
explicitly. 
The rest of the `Bootstrap.groovy` looks like this (Note the conditional check 
for Environment - Very important!) 

{% highlight java %}
import grails.util.Environment
import org.codehaus.groovy.grails.orm.hibernate.cfg.GrailsDomainBinder 

class BootStrap { 

    def init = { servletContext ->
        if (Environment.current == Environment.TEST) { 
            disableAutoTimestamp(YourDomain.class) 
        } 
    } 

    def disableAutoTimestamp(Class domClass) { 
        ... 
    } 
} 
{% endhighlight %}

## References

1. [http://stackoverflow.com/questions/28735133/pull-domain-mapping-in-bootstrap-and-modify-it-grails](http://stackoverflow.com/questions/28735133/pull-domain-mapping-in-bootstrap-and-modify-it-grails)
2. [http://grails.github.io/grails-doc/2.5.0/ref/Database%20Mapping/autoTimestamp.html](http://grails.github.io/grails-doc/2.5.0/ref/Database%20Mapping/autoTimestamp.html)
3. [http://grails.1312388.n4.nabble.com/Grails-2-3-GrailsDomainBinder-getMapping-no-longer-static-tp4648984p4648994.html](http://grails.1312388.n4.nabble.com/Grails-2-3-GrailsDomainBinder-getMapping-no-longer-static-tp4648984p4648994.html)
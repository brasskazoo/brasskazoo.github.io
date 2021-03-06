---
layout: post
title: How to load properties files into Spring and expose to the Java classes
date: '2010-03-01T15:00:00.000+11:00'
categories: software-development

author: brasskazoo
tags:
- Java
- Software Development
- Spring
modified_time: '2012-01-02T19:29:28.102+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-8438898235180446981
blogger_orig_url: http://blog.brasskazoo.com/2010/02/how-to-load-properties-files-into.html
---

**The issue**:

>Our webapp is using a spring and hibernate backend, and both
spring configuration and Java classes need to access database connection 
properties.

Currently there is an
`application.properties` file and a spring `applicationContext.xml`, both
which contain the same information for the database connection.  Ideally, 
there should be only one configuration file to define the properties, which is 
used by both spring and Java.</blockquote>Spring has the ability to load 
properties files as part of the application configuration, for use internally. 
The properties can be referenced within spring configuration files using 
ant-style placeholders, e.g. `${app.var}`.  To solve our issue though we'll
also need to provide a populated bean that can be used by our production 
classes.  Here's the properties file (`application.properties`):

````
appl.name=My Web Application
appl.home=/Users/webapp/application/ 

# Database properties 
db.driver=org.hsqldb.jdbcDriver 
db.name=v8max.db 
db.url=jdbc:hsqldb:file://${appl.home}/database/${db.name} 
db.user=SA 
db.pass=
````

Spring loads the properties file using a bean of the type
`org.springframework.beans.factory.config.PropertyPlaceholderConfigurer`, and then we can reference the properties directly in the spring configuration
file.  Note that we can also use placeholders within the properties file, as 
in the db.url above - the `PropertyPlaceholderConfigurer` will resolve them too!

{% highlight xml %}
<!-- Load in application properties reference -->
    <bean id="applicationProperties"
class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location"
value="classpath:application.properties"/>
    </bean>
    <bean id="dataSource"
class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="${db.driver}"/>
        <property name="url" value="${db.url}"/>
        <property name="username" value="${db.user}"/>
        <property name="password" value="${db.pass}"/>
    </bean>
{% endhighlight %}

So now we've removed the need to have literals in the spring config! 

## Exposing the spring properties bean in java

To allow our Java classes to
access the properties from the same object as spring, we'll need to extend the 
`PropertyPlaceholderConfigurer` so that we can provide a more convenient method
for retrieving the properties (there is no direct method of retrieving 
properties!).  We can extend the spring provided class to allow us to reuse 
spring's property resolver in our Java classes: 

{% highlight java %}
public class PropertiesUtil extends PropertyPlaceholderConfigurer {
   private static Map propertiesMap; 

   @Override 
   protected void processProperties(ConfigurableListableBeanFactory 
beanFactory, 
             Properties props) throws BeansException { 
        super.processProperties(beanFactory, props); 

        propertiesMap = new HashMap<string, string="">(); 
        for (Object key : props.keySet()) { 
            String keyStr = key.toString(); 
            propertiesMap.put(keyStr, 
parseStringValue(props.getProperty(keyStr), 
                props, new HashSet())); 
        } 
    } 

    public static String getProperty(String name) { 
        return propertiesMap.get(name); 
    } 
} 
{% endhighlight %}

If we now update the `applicationProperties` bean to use the `PropertiesUtil`
class, we can use the static getProperty method to access the resolved 
properties via the same object as the spring configuration bean.  Of course, 
we could run into problems if a class tries to use `PropertiesUtil` before the
spring context has been initialised. For example if you're registering 
`ServletContextListeners` in your `web.xml` before configuring spring, you'll get
a `NullPointerException` if one of those classes tries to use `PropertiesUtil`.
For that reason I've had to declare the spring context before any context 
listeners. 

{% highlight xml %}
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        /WEB-INF/applicationContext.xml
    </param-value>
</context-param>

<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>

<!-- Other listeners -->
{% endhighlight %}

## References 
1. 
[http://www.jdocs.com/spring/1.2.8/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html](http://www.jdocs.com/spring/1.2.8/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html) 
1. 
[http://j2eecookbook.blogspot.com/2007/07/accessing-properties-loaded-via-spring.html](http://j2eecookbook.blogspot.com/2007/07/accessing-properties-loaded-via-spring.html) 
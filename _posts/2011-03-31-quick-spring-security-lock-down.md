---
layout: post
title: A Quick Spring Security Lock-Down
date: '2011-03-31T14:00:00.000+11:00'
categories: software-development web-development

author: brasskazoo
tags:
- Java
- Security
- JAAS
- Maven
- Spring
modified_time: '2012-01-02T19:28:34.662+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-734247131339571086
blogger_orig_url: http://blog.brasskazoo.com/2009/03/quick-spring-security-lock-down.html
---

My new shiny web application is fantastically useful, but only to a certain 
group of people (i.e. my team), and should only be accessible by them. 

So, before being able to put it into real production, I needed a security 
framework around it. 

A legacy 
[JAAS](http://download.oracle.com/javase/6/docs/technotes/guides/security/jaas/JAASRefGuide.html) 
component of ours exists, but given my application was making use of the 
Spring framework, I compared Spring's offering to the JAAS infrastructure. 

[Popular](http://java.sys-con.com/node/1002315) 
[opinion](http://stackoverflow.com/questions/628416/jaas-for-human-beings/694820#694820) 
seems to be that JAAS was build for J2SE, not J2EE, and is designed for things 
at a much 'lower level' than web applications, such as client-side applets 
rather than server-side applications. 

## Maven

First things first: Maven dependencies.

I'm using `spring-webmvc 2.5.6`, so I'd like to get security working with the 
application as it stands now - the latest pre-3.0 release of spring-security 
is 2.0.6-RELEASE: 

{% highlight xml %}
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>2.5.6</version>
</dependency>
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-core</artifactId>
  <version>2.0.6.RELEASE</version>
</dependency>
{% endhighlight %}

## Web Context

The web context requires two things:

<ol>
 <li>Context location

{% highlight xml %}
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>
/WEB-INF/applicationContext.xml 
/WEB-INF/applicationContext-security.xml 
  </param-value>
</context-param>
{% endhighlight %}

(we'll create the security context in the next step) 

 </li>
 <li>Filter definition

{% highlight xml %}
<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}

The url-pattern will mean all requests pass through the filter (which will 
have more explicit criteria).
</li>
</ol>

## Security Context

Now we get to the real meat of the security layer!

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>

<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-2.0.1.xsd">

    <http auto-config="true">
        <intercept-url pattern="/**" access="ROLE_USER" />
        <http-basic />
    </http>

    <authentication-provider>
        <password-encoder hash="md5"/>
        <user-service>
            <user name="user"
            password="aabbccddeeff001122334455667788ff"
            authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</beans:beans>
{% endhighlight %}

Here we can see the configuration for http requests. The 'auto-config' sets 
the defaults (refer to the doco in the references), which are overridden by 
the contents of the tag. We'll let in one user for now with the role 
'ROLE_USER', defined in the `authentication-provider` section. 

Including `http-basic` just puts the preference on using the basic HTTP 
prompt, but removing that line would use Spring's default login page (with 
user/pass and 'remember me' checkbox). 

And its done! Deploying the application and loading the page demands a login 
before progressing. 

## Next
Future improvements might involve setting up a styled login page,
hooking up an LDAP connection (but with restrictions). 
Oh, and Selenium tests..! 

## References 
1. Spring Source, _Spring Security Reference Documentation_
[http://static.springsource.org/spring-security/site/docs/2.0.x/reference/ns-config.htm](http://static.springsource.org/spring-security/site/docs/2.0.x/reference/ns-config.htm)
1. Peter Mularien, _5 Minute Guide to Spring Security_
[http://www.mularien.com/blog/2008/07/07/5-minute-guide-to-spring-security/](http://www.mularien.com/blog/2008/07/07/5-minute-guide-to-spring-security/)
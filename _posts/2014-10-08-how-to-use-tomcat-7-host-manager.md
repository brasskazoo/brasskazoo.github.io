---
layout: post
title: How to use Tomcat 7 Host Manager
date: '2014-10-08T22:33:00.000+11:00'
categories: software-development web-applications tomcat

author: brasskazoo
tags:
- Apache
- Web Applications
- tomcat
modified_time: '2015-06-11T17:11:33.350+10:00'
thumbnail: http://2.bp.blogspot.com/-jCdp6LQXKow/VWxUC-HQE_I/AAAAAAAAAG0/T3eXFJGUdhg/s72-c/tomcat-host-manager-add.png
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-6013224006658541829
blogger_orig_url: http://blog.brasskazoo.com/2014/10/how-to-use-tomcat-7-host-manager.html
---

**Tomcat Host Manager** is a web application inside of Tomcat that manages _Virtual Hosts_ within the
Tomcat application server.

A _Virtual Host_ 
allows you to define multiple hostnames on a single server, so you can use the 
same server to handles requests to different subdomains, for example, `ren.myserver.com` 
and `stimpy.myserver.com`.

Unfortunately documentation on the GUI side 
of the Host Manager doesn't appear to exist, but documentation on configuring 
the virtual hosts manually in `context.xml` is here:

[http://tomcat.apache.org/tomcat-7.0-doc/virtual-hosting-howto.html](http://tomcat.apache.org/tomcat-7.0-doc/virtual-hosting-howto.html).

The full explanation of the `Host` parameters you can find here:

[http://tomcat.apache.org/tomcat-7.0-doc/config/host.html](http://tomcat.apache.org/tomcat-7.0-doc/config/host.html).<h2 
style="background-color: white; border: 0px; clear: both; color: #222222; 
font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 15px; 
line-height: 19.5px; margin-bottom: 1em; padding: 0px;">**Accessing the 
Virtual Host Manager**</h2>

To access the virtual host manager, you need to have appropriate 
permissions set up in your `tomcat-users.xml` - at the very least the 
`manager-gui` role (see the official documentation 
[here](http://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html#Configuring_Manager_Application_Access) 
for full information). It should look something like this:

{% highlight xml %}
<role rolename="manager-gui"/>
  <user username="tomcat-admin" password="change-me!"
roles="manager-gui"/>
</role>
{% endhighlight %}

Once you have access, you can access the host-manager application at
[http://localhost:8080/host-manager](http://localhost:8080/host-manager).

## Adding a new virtual host

The *Add Virtual Host* panel looks like this:

[<img border="0" height="101"
src="http://2.bp.blogspot.com/-jCdp6LQXKow/VWxUC-HQE_I/AAAAAAAAAG0/T3eXFJGUdhg/s320/tomcat-host-manager-add.png" 
width="320" 
/>](http://2.bp.blogspot.com/-jCdp6LQXKow/VWxUC-HQE_I/AAAAAAAAAG0/T3eXFJGUdhg/s1600/tomcat-host-manager-add.png)

Tomcat Virtual Host Manager

At a minimum you need 
the `Name` and `App Base` fields defined. From those, Tomcat 
will create the following directories: 

{% highlight java %}
{CATALINA_HOME}\conf\Catalina\{Name} 
{CATALINA_HOME}\{App Base} 
{% endhighlight %}

`App Base` will be where web applications will be deployed to the virtual host. Can be relative
or absolute.

`Name` is usually the fully-qualified domain name (e.g. `ren.myserver.com`)

`Alias` can be used to extend the `Name` also where 
two addresses should resolve to the same host (e.g. `www.ren.myserver.com`). Note that this needs to be reflected 
in DNS records.

I'll let you look up the other manual parameters, but I find the most useful ones are:

`Unpack WARs`: 
Unpack WAR files placed or uploaded to the App Base, as opposed to running 
them directly from the WAR.

`Auto Deploy`: Automatically redeploy applications placed into App Base.
Careful - this is potentially dangerous for Production environments!

`Deploy On Startup`: Automatically boot up applications under App Base when Tomcat
starts

`Copy XML`: Copy 
an application's `META-INF/context.xml` to the App Base/XML Base on deployment, 
and use that exclusively, regardless of whether the application is updated. 
Irrelevant if `Deploy XML` is false (*Tomcat 8 only*).

`Deploy XML`: 
Determines whether to parse the application's `/META-INF/context.xml`

`Manager App`: Add 
the manager application to the Virtual Host (Useful for controlling the 
applications you might have underneath `ren.myserver.com`). You would then access this 
at `http://{Name}/manager`

Once you hit *Add*, Tomcat will create the directories above and move in 
the tomcat-manager app config under `{CATALINA_HOME}\conf\Catalina\{Name}`. 
You should be able to hit that straight away!

## Now, a warning..

[<img border="0" 
src="http://33.media.tumblr.com/1b92aae859e06f19578f1af074e2e0a8/tumblr_inline_mjg1wt3KaL1qz4rgp.gif" 
height="165" width="320" 
/>](http://33.media.tumblr.com/1b92aae859e06f19578f1af074e2e0a8/tumblr_inline_mjg1wt3KaL1qz4rgp.gif)

You need to add your virtual-host entry to Tomcat's `server.xml` for it to persist after a
restart! (See the documentation link at the start of the article for how to
do this)

For who-knows-what reason, the virtual-host entry you add via the GUI isn't
written to disk. So, you may ask yourself, why don't I just add the entry into 
`server.xml` instead of trying to use the GUI? Good question.. 

 
<small>*Originally posted by myself as an answer on
[stackoverflow.com](http://stackoverflow.com/a/26248511/6340).*</small>
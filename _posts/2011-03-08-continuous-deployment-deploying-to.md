---
layout: post
title: 'Continuous Deployment: Deploying to Glassfish with Maven and TeamCity'
date: '2011-03-08T14:25:00.000+11:00'
categories: software-development continuous-integration

author: brasskazoo
tags:
- Java
- Continuous Integration
- Glassfish
- Maven
- Software Development
- TeamCity
modified_time: '2012-01-02T19:28:58.378+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-2347352711078141555
blogger_orig_url: http://blog.brasskazoo.com/2011/03/continuous-deployment-deploying-to.html
---

The sooner we can deploy a tested and verified piece of software, the better!
Here I'm describing an automated deployment process that uses Maven to deploy 
to a Glassfish application server. [TeamCity](https://www.jetbrains.com/teamcity) facilitates the build and test
stages, with an additional deployment of the packaged web application. Using 
the glassfish plugin for maven 2, we can integrate the application server 
deployment into the continuous integration cycle, and provide a constantly 
up-to-date development/test environment.

##  Glassfish

I'll create a new domain from scratch for the maven apps, using
the default port values (i.e. admin port 4848, http port 8080), but setting 
the admin password and master password.  In setting up the glassfish domain we 
generate a password file so that we are not storing any passwords in plain 
text - such as in the pom or settings.xml 

````
cd ${glassfish.home}/bin
./asadmin create-domain --savemasterpassword=true my-apps
````

The _`--savemasterpassword`_ switch generates an encrypted
'master-password' file in the domains/my-apps directory. 
##    Maven
Maven profiles makes it easy to have machine-specific variables so
that moving to other platforms in the future is straightforward.  On the on 
the host machine I've created the file `~/.m2/settings.xml`.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <profiles>
    <profile>
      <id>glassfish-context</id>
      <properties>
        <local.glassfish.home>/Users/brass/bin/glassfishv3</local.glassfish.home>
        <local.glassfish.user>admin</local.glassfish.user>
        <local.glassfish.domain>my-apps</local.glassfish.domain>
        <local.glassfish.passfile>
        ${local.glassfish.home}/glassfish/domains/${local.glassfish.domain}/master-password
        </local.glassfish.passfile>
      </properties>
    </profile>
  </profiles>

  <activeProfiles>
    <activeProfile>glassfish-context</activeProfile>
  </activeProfiles>
</settings>
{% endhighlight %}

This gives us the parameters for the glassfish instance that we will use in 
our pom.  **Update:** _I found that while the above works for v3.1, my dev
machine's glassfish v3.0.1 needed to reference glassfish home one directory 
deeper:_

{% highlight xml %}
<local.glassfish.home>/Users/brass/bin/glassfishv3/glassfish 
</local.glassfish.home> 
... 
<local.glassfish.passfile> 
${local.glassfish.home}/domains/${local.glassfish.domain}/master-password 
</local.glassfish.passfile> 
{% endhighlight %}

In the project _pom.xml_, we define a profile for glassfish deployment:

{% highlight xml %}
<profile>
  <id>glassfish-deploy</id>
  <pluginRepositories>
    <pluginRepository>
      <id>maven.java.net</id>
        <name>Java.net Maven2 Repository</name>
        <url>http://download.java.net/maven/2</url>
      </pluginRepository>
    </pluginRepositories>
    <build>
      <plugins>
      <plugin>
        <groupId>org.glassfish.maven.plugin</groupId>
        <artifactId>maven-glassfish-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <glassfishDirectory>${local.glassfish.home}</glassfishDirectory>
          <user>${local.glassfish.user}</user>
          <passwordFile>${local.glassfish.passfile}</passwordFile>
          <autoCreate>true</autoCreate>
          <debug>true</debug>
          <echo>false</echo>
          <terse>true</terse>
          <domain>
            <name>${local.glassfish.domain}</name>
            <adminPort>4848</adminPort>
            <httpPort>8080</httpPort>
            <httpsPort>8443</httpsPort>
            <iiopPort>3700</iiopPort>
            <jmsPort>7676</jmsPort>
            <reuse>false</reuse>
          </domain>
          <components>
            <component>
              <name>${project.artifactId}</name>
              <artifact>${project.build.directory}/${project.build.finalName}.war</artifact>
            </component>
          </components>
        </configuration>
      </plugin>
    </plugins>
  </build>
</profile>
{% endhighlight %}

The repository for the glassfish plugin repository is specified within the 
profile since its not part of the larger project in this case.  As you see the 
variables from the local settings.xml are used for the glassfish config. 

##  TeamCity

The TeamCity setup needs to include two things in the maven2 runner config:

1. Glassfish goals
1. Profile parameters

<img alt="TeamCity - maven runner configuration" class="alignnone size-full wp-image-333" height="374" src="http://codingbone.files.wordpress.com/2011/03/picture-4.png" title="TeamCity-glassfishdeploy" width="500" />

The glassfish goals that are used should be able to start the domain if it 
isn't running, and replace the application.  The 'redeploy' goal would allow a 
hot-swap deployment, if for example we were running other applications on the 
domain.  See the plugin page 
([http://maven-glassfish-plugin.java.net/](http://maven-glassfish-plugin.java.net/)) 
for more info. 

## Next

So now this process provides us with continuous deployment - a commit
will be built, tested and deployed automatically, allowing changes to the 
software to be seen and used almost immediately! 
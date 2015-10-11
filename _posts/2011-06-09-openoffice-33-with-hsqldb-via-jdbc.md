---
layout: post
title: OpenOffice 3.3 with HSQLDB via JDBC
date: '2011-06-09T17:07:00.000+10:00'
categories: applications

author: brasskazoo
tags:
- database
- hsqldb
- OpenOffice
- JDBC
modified_time: '2012-01-02T19:27:54.809+11:00'
thumbnail: http://3.bp.blogspot.com/-PO0ALgVuGy8/TssMSsRbINI/AAAAAAAAABw/zcWbtT3WUQw/s72-c/OO-Java1.png
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-4416989808933344083
blogger_orig_url: http://blog.brasskazoo.com/2011/06/openoffice-33-with-hsqldb-via-jdbc.html
---

I had developed an application that stored some metrics in a 
[HyperSQL](http://hsqldb.org/) database for offline processing and reporting with OpenOffice.
It seemed simple enough - OpenOffice would connect to the database via JDBC, 
with the JDBC URL and driver class..trouble was, no matter what I did, the 
driver class would not be recognised! 
After endlessly searching on the web for a solution, I realised the piece of 
the puzzle I was missing - the version of the hsqldb jar file. The version 2.0.0 jar appears to be incompatible with OpenOffice 3.3 - Reverting back to 1.8.0.10 solves the problem, and the JDBC driver class is recognised.
So I'll publish the process in full here, just in case anyone else runs into 
the same problem!

## Components
1. OpenOffice 3.3.0 (The latest stable at the time of writing)
1. hsqldb-1.8.0.10.jar 
1. An existing hsql database (created using hsqldb 1.8.0.10 jars) 

## OpenOffice Configuration
Despite OpenOffice using hsqldb for it's internal
databases, the driver is not natively available for JDBC. 
Under `Preferences > OpenOffice.org > Java`, ensure that a JRE is
registered. Open up the `classpath` dialog, and add an archive entry for the 
hsqldb jar: 

<img border="0" height="169" alt="OpenOffice Java and Classpath Configuration" src="http://3.bp.blogspot.com/-PO0ALgVuGy8/TssMSsRbINI/AAAAAAAAABw/zcWbtT3WUQw/s320/OO-Java1.png" width="320" />

This is basically where I had been running into trouble. My application originally used hsqldb version 2.0.0 with Hibernate. Despite the
driver class being the same (`org.hsqldb.jdbcDriver`), it seems OpenOffice would not recognise this version of hsqldb.

##     Connect OpenOffice to Database

Using OpenOffice's Database, select
`Connect to an existing database` (using JDBC). 
Enter the JDBC Connection details: 
1. URL: The JDBC URL minus the 'jdbc:' - should be of the form 
`hsqldb:/path/to/database` - also append `;default_schema=true` to the end. 
1. JDBC driver class is: `org.hsqldb.jdbcDriver` 
Note that 'database' is the collective database name for the files making up 
the hsqldb instance (e.g. database.log, database.properties, database.script). 

<img border="0" height="209" alt="HSQLDB JDBS Connection" src="http://1.bp.blogspot.com/-549K6qCN-Ik/TssMyvHRhGI/AAAAAAAAAB4/i8Jj4-EeFEs/s320/OO-Java2.png" width="320" />

Click 'Next', enter the default username of `sa` (No password required).
Accept the defaults for registering the database, choose a location to save 
and you're done! 

If anyone has an explanation of the OpenOffice incompatibility, please
leave a comment! 
---
layout: post
title: Integration testing of a custom JIRA CreateOrCommentHandler email processor
date: '2011-11-22T14:18:00.001+11:00'
author: brasskazoo
categories: software-development atlassian-jira code-quality

tags:
- Apache James
- Java
- Continuous Integration
- Code Quality
- junit
modified_time: '2011-12-06T15:33:48.811+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-1765605058451997591
---

I'm working on a customised email handler (based on Atlassian's original [CreateOrCommentHandler](http://docs.atlassian.com/software/jira/docs/api/latest/com/atlassian/jira/service/util/handler/CreateOrCommentHandler.html) class), that extends some of the available parameters to enable our team to better handle their Jira loads.

I won't go into too much detail about the handler itself, but I would like to document the test harness that allows me to perform some integration/regression testing against it.

The test environment includes:

* an instance of JIRA (with some manual modifications)
* an IMAP/SMTP server (Apache James).

## JIRA Environment

An email message handler is not a normal plugin module type for JIRA (see the list [here](http://confluence.atlassian.com/display/JIRA043/JIRA+Plugin+Guide#JIRAPluginGuide-JIRAPluginModuleTypes")), so there is no entry in the
`atlassian-plugin.xml` required for it. The plugin infrastructure in this case is merely a vehicle to upload the classes into Jira.

## IMAP Service

We need to manually enable access to the custom handler by editing the
`imapservice.xml` file (under `WEB-INF/classes/services/com/atlassian/jira/service/services/imap/imapservice.xml`) to include our class:

{% highlight xml %}
...
<property>
    <key>handler</key>
    <name>admin.service.common.handler</name>
    <type>select</type>
    <values>
        <value>
            <key>com.example.CreateOrCommentHandler</key>
            <value>Custom CreateOrCommentHandler</value>
        </value>
        <value>
            <key>com.example.CreateIssueHandler</key>
            <value>Custom CreateIssueHandler</value>
        </value>
        ...
        (Additional handlers go here!)
        ...
    </values>
</property>
...
{% endhighlight %}

Once this is done (and JIRA is restarted), we can add services that use this handler by adding a regular IMAPService, and selecting our custom handler.



## JIRA Configuration

We also need to set JIRA up with some dummy data - e.g. projects, users - and export this configuration as a testing baseline (using the
<a href="http://confluence.atlassian.com/display/JIRA044/Backing+Up+Data#BackingUpData-UsingJIRAsXMLbackuputility">administration tool</a>).

## IMAP/SMTP Server

Can't really test an email handling plugin without an email!

Rather than mocking out a whole bunch of complex classes (the message itself _and_ JIRA's internals), using a lightweight mail server and simulating a real environment would be much simpler.

I found [Apache James](http://james.apache.org/), and was able to quickly get it up and running as a standalone server (To use the default ports however - SMTP: 25, IMAP: 143 -, you need to start the server as root). In the default configuration, James will create a user account for the receiver(s) when receiving an email, if they don't already exist. So we are free to send emails to and check the accounts of which ever user we want, without a setup overhead!

## Email Client

A utility class will help us to send emails and clear user's inboxes (this class based on Claude Duguay's
<a href="http://www.ibm.com/developerworks/java/library/j-james1/index.html">IBM article</a>).

The parent class
<code>javax.mail.Authenticator</code> helps us with the <code>javax.mail.Session</code> connection. 


{% highlight java %}
public class JamesMailClient extends Authenticator {

    public static final int SHOW_MESSAGES = 1;

    public static final int CLEAR_MESSAGES = 2;

    public static final int SHOW_AND_CLEAR = SHOW_MESSAGES + CLEAR_MESSAGES;


    private Session _session;

    private PasswordAuthentication _passwordAuthentication;

    private String _userAddress;

    private String _user;

    private String _pass;

    private String _host;


    public JamesMailClient(String user, final String pass, String host, boolean debug) {
        _user = user;
        _host = host;
        _userAddress = _user + '@' + _host;

        _pass = pass;
        _passwordAuthentication = new PasswordAuthentication(user, _pass);

        Properties props = new Properties();
        props.put("mail.user", user);
        props.put("mail.host", host);
        props.put("mail.debug", debug ? "true" : "false");
        props.put("mail.store.protocol", "imap");
        props.put("mail.transport.protocol", "smtp");

        _session = Session.getInstance(props, this);
    }

    public void checkInbox(int mode) throws MessagingException, IOException {..}

    public void sendMessage(String recipientTo, String recipientCC, String messageSubject, String messageBody) throws MessagingException {..}

    public boolean waitForIncomingEmail(long timeout, int emailCount) throws MessagingException, InterruptedException {..}
}
{% endhighlight %}

## Test class structure

The test classes extend Atlassian's <code>com.atlassian.jira.functest.framework.FuncTestCase</code>, which provides us with access to&nbsp;convenience&nbsp;functions to restore data from XML and navigate JIRA easily.


The test harness is handled by overridding `setUpTest()` and `tearDownTest()` (Note that to be a 'good neighbour', I restored a 'clean' version of the config when we're done):

{% highlight java %}
@Override                                                                        ˚
protected void setUpTest() {
    super.setUpTest();

    mailUserAccount = new JamesMailClient("user", "user", "localhost", false);
    _mailBugsAccount = new JamesMailClient("bugs", "bugs", "localhost", false);

    try {
        _mailUserAccount.checkInbox(JamesMailClient.CLEARMESSAGES);
        mailBugsAccount.checkInbox(JamesMailClient.CLEARMESSAGES);
    } catch (MessagingException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }

    //Import some test data.
    administration.restoreData("jira-test-base.xml");
}

@Override
protected void tearDownTest() {
    super.tearDownTest();

    administration.restoreData("jira-clean-install.xml");
    try {
        mailBugsAccount.checkInbox(JamesMailClient.CLEARMESSAGES);
        mailUserAccount.checkInbox(JamesMailClient.CLEARMESSAGES);
    } catch (Exception e) {
        e.printStackTrace();
    }
}

{% endhighlight %}


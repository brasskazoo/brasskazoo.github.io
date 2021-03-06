---
layout: post
title: Using an email server in your unit testing
date: '2011-12-06T13:33:00.001+11:00'
categories: software-development code-quality

author: brasskazoo
tags:
- Apache James
- Java
- unit testing
- integration testing
- Software Development
- Code Quality
- junit
modified_time: '2012-01-02T19:26:01.137+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-8536531332904773416
blogger_orig_url: http://blog.brasskazoo.com/2011/12/how-to-use-interact-with-apache-james.html
---

## How to interact with Apache James in integration tests
At times, I have the
need for my development testing to interact with an email server (e.g. testing 
[cgi-bin 
scripts](/2011/10/using-webdriver-jbehave-to-test-dynamic.html),
[JIRA](http://www.atlassian.com/software/jira) mail plug-ins), to verify that 
emails are sent and received by either my components or external systems. 
I found that [Apache James](http://james.apache.org/) was quick to get up and 
running, with very little configuration.

Apache James is a open source Java implementation of a fully-featured mail 
server. It is component based, so you can attach or detach what ever parts you 
like (e.g. I am only interested in the SMTP and IMAP connectors), and for my 
purposes requires very little setup (although I wouldn't recommend using 
default settings in a permanent production environment).

Note that to use the default ports (SMTP: 25, IMAP: 143), you'll have to run 
as root on unix, or administrator on Windows.

James can automatically create a user account for the recipients(s) when 
receiving an email, if they don't already exist. So we are free to send emails 
to and check the accounts of which ever user we want, without a setup 
overhead! 

##   Mail Client Utility

When the James server is running, a utility class will
help us to send emails and clear user's inboxes (this class based on Claude 
Duguay's [IBM 
article](http://www.ibm.com/developerworks/java/library/j-james1/index.html)). 
The parent class `javax.mail.Authenticator` helps us with the 
`javax.mail.Session` connection. 

{% highlight java %}
public class JamesMailClient extends Authenticator {
    public static final int SHOW_MESSAGES = 1;
    public static final int CLEAR_MESSAGES = 2;
    public static final int SHOW_AND_CLEAR = SHOW_MESSAGES + CLEAR_MESSAGES;

    private final Session _session;
    private final PasswordAuthentication _passwordAuthentication;
    private final String _userAddress;
    private final String _user;
    private final String _pass;
    private final String _host;

    public JamesMailClient(final String user, final String pass,
            final String host, final boolean debug) {
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

    public void sendMessage(final String recipientTo,
            final String messageSubject, final String messageBody)
            throws MessagingException {
        Properties properties = new Properties();
        properties.put("mail.smtp.host", _host);
        properties.put("mail.smtp.port", "25");
        properties.put("mail.smtp.username", _user);
        properties.put("mail.smtp.password", _pass);
        Session session = Session.getDefaultInstance(properties, null);

        Message msg = new MimeMessage(session);
        msg.addFrom(new Address[]{new InternetAddress(_userAddress)});
        msg.setRecipients(Message.RecipientType.TO,
            parse(recipientTo));
        msg.setSubject(messageSubject);
        msg.setText(messageBody);
        Transport.send(msg);
    }

    public boolean waitForIncomingEmail(final long timeout,
            final int emailCount)
            throws MessagingException, InterruptedException {
        Folder inbox = connect(Folder.READ_ONLY);

        Message[] msgs = inbox.getMessages();

        long t0 = System.currentTimeMillis();
        while (msgs.length < emailCount) {
            Thread.sleep(timeout / 10);
            if ((System.currentTimeMillis() - t0) > timeout) {
                return false;
            }
            msgs = inbox.getMessages();
        }

        disconnect(inbox, false);
        return true;
    }

    public void checkInbox(final int mode)
            throws MessagingException, IOException {
        if (mode == 0) {
            return;
        }

        boolean show = (mode & SHOW_MESSAGES) > 0;
        boolean clear = (mode & CLEAR_MESSAGES) > 0;
        String action = (show ? "Show" : "")
            + (show && clear ? " and " : "") + (clear ? "Clear" : "");

        System.out.println(action + " INBOX for " + _userAddress);

        Folder inbox = connect(Folder.READ_WRITE);

        Message[] msgs = inbox.getMessages();
        if (msgs.length == 0 && show) {
            System.out.println("No messages in inbox");
        } else {
            System.out.println(msgs.length + " messages in inbox");
        }

        for (Message msg1 : msgs) {
            MimeMessage msg = (MimeMessage) msg1;
            if (show) {
                System.out.println("    From: " + msg.getFrom()[0]);
                System.out.println(" Subject: " + msg.getSubject());
                System.out.println(" Content: " + msg.getContent());
            }
            if (clear) {
                msg.setFlag(Flags.Flag.DELETED, true);
            }
        }
        disconnect(inbox, true);
    }

    public Message getMessage(final int index) throws MessagingException {
        Folder inbox = connect(Folder.READ_WRITE);

        Message[] msgs = inbox.getMessages();

        disconnect(inbox, false);
        return msgs[index];
    }

    public int getMessageCount() throws MessagingException {
        Folder inbox = connect(Folder.READ_ONLY);

        Message[] msgs = inbox.getMessages();

        disconnect(inbox, false);
        return msgs.length;
    }

    public PasswordAuthentication getPasswordAuthentication() {
        return _passwordAuthentication;
    }

    private Folder connect(final int accessType) throws MessagingException {
        Store store = _session.getStore();
        store.connect();

        Folder root = store.getDefaultFolder();
        Folder inbox = root.getFolder("inbox");
        inbox.open(accessType);
        return inbox;
    }

    private void disconnect(final Folder inbox, final boolean expunge)
            throws MessagingException {
        final Store store = inbox.getStore();
        inbox.close(expunge);
        store.close();
    }
} 
{% endhighlight %}

This utility class gives us the ability to send and receive messages, as well 
as providing a method of pausing in a mailbox until a new message arrives. 
You'll notice the constructor includes a username and password, although the 
default behaviour of James when it receives an email for an account that 
doesn't exist is to create a new account with a password that is the same as 
the username. The host parameter will most likely be `localhost` unless you've 
deployed James to another server. 
The `waitForIncomingEmail` method allows us to pause execution while waiting 
for an email to be received, useful if the next test stage requires an email 
to be present in the inbox. 
We can also clear out an inbox using `checkInbox` with the constant 
`CLEAR_MESSAGES`.

##   Test class setup

In a JUnit test class, we can set up references to user
accounts using instances of `JamesMailClient` and clear out the inboxes when 
we've finished. 
If the account doesn't exists (i.e. before you've sent an email to the account 
or created it manually), a `NoSuchProviderException` would be thrown when the 
account is accessed in `tearDown()`. 

{% highlight java %}
    @Before 
    protected void setUp() { 
        _mailFooAccount = new JamesMailClient("foo", "foo", "localhost", 
false); 
        _mailBarAccount = new JamesMailClient("bar", "bar", "localhost", 
false); 
    } 
    @After 
    public void tearDown() { 
        try { 
            _mailFooAccount.checkInbox(JamesMailClient.CLEAR_MESSAGES); 
            _mailBarAccount.checkInbox(JamesMailClient.CLEAR_MESSAGES); 
        } catch (MessagingException e) { 
            e.printStackTrace(); 
        } catch (IOException e) { 
            e.printStackTrace(); 
        } 
    } 
{% endhighlight %}

Our test harness is then ready for test cases to send and receive emails! 

##   Test Case Usage

Using the functions in `JamesMailClient`, this code shows
`foo@localhost` sending an email to `bar@localhost`, and `bar` waiting to 
receive it! 

{% highlight java %}
    final String subject = "Neque porro quisquam dolor sit amet: " 
            + System.currentTimeMillis(); 
    final String body = "Lorem ipsum dolor sit amet, consectetur " 
            + "adipiscing elit. Nullam a elit purus, eget " 
            + "eleifend turpis. Suspendisse condimentum dictum."; 

     _mailFooAccount.sendMessage("bar@localhost", subject, body); 
    // Delay to allow for message delivery 
    _mailBarAccount.waitForIncomingEmail(1000, 1); 

    assertEquals(subject, _mailBarAccount.getMessage(0).getSubject); 
{% endhighlight %}

##  Apache James & Maven

So far I've been simply running Apache James from
the command line, but it does exist in the maven repository 
([http://repo1.maven.org/maven2/org/apache/james/](http://repo1.maven.org/maven2/org/apache/james/)). 
In the future I could be tempted to write a maven plugin or similar for it to 
launch for integration testing targets.. Stay tuned! 
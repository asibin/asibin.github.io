---
layout: post
title:  "Setting up Nagios with Postfix and Gmail"
date:   2016-02-27 23:00:00
categories: nagios postfix gmail tutorial
tags: [nagios, postfix, gmail, relay, notification, tutorial, install, howto, ubuntu]
---

This is just a part of configuration listing some of the steps that i thought needed writing down. 

## Postfix setup

In order to use postfix and Gmail to relay localhost mail you need to install

{% highlight tcsh %}
sudo apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules
{% endhighlight %}

while installing Postfix will ask about some settings:

- choose `Internet Site`
- choose domain name for unqualified mail addresses for example ``sib.in.rs``
- allow networks should be kept at 127.0.0.1 if you want local relay or you can specify list of IPs or networks from which to allow relay
- mailbox quota leave at `0`
- leave `+` as is

After installation is over go to Postfix's config:

{% highlight tcsh %}
sudo nano /etc/postfix/main.cf
{% endhighlight %}

and add this to the end:

{% highlight tcsh %}
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/postfix/cacert.pem
smtp_use_tls = yes
{% endhighlight %}

Now we need to define username and password for gmail, edit this file:

{% highlight tcsh %}
sudo nano /etc/postfix/sasl_passwd
{% endhighlight %}

And add the following line:

{% highlight tcsh %}
[smtp.gmail.com]:587    USERNAME@gmail.com:PASSWORD
{% endhighlight %}

If you are using google apps change @gmail to @<your_domain>.

Once you have entered your username / password run `postmap` to create postfix lookup table:

{% highlight tcsh %}
postmap /etc/postfix/sasl/sasl_passwd
{% endhighlight %}

This will create `sasl_passd.db` lookup table in same directory.

Next make sure that `sasl_passwd` and `sasl_passd.db` files have the right permissions:

{% highlight tcsh %}

chown -R root:postfix /etc/postfix/sasl
chmod 750 /etc/postfix/sasl
chmod 640 /etc/postfix/sasl/sasl_passwd*
{% endhighlight %}

Restart postfix:

{% highlight tcsh %}
sudo service postfix restart
{% endhighlight %}

And test if email relays are working:

{% highlight tcsh %}
echo "Tested" | mail -s "Testing Gmail Relay" user@example.net
{% endhighlight %}

replace user@example.net with some of the users defined in `/etc/aliases` and domain you defined while installing postfix.

**Note:** If you edit `/etc/aliases` you need to issue `sudo newaliases` in order to load new aliases.

## Nagios setup

First define contacts that you wish to notify and their email addresses in:

{% highlight tcsh %}
sudo nano /usr/local/nagios/etc/objects/contacts.cfg
{% endhighlight %}

In order to send email with Nagios i had to change mail command used for sending email as it was `sendmail` for some reason which doesn't accept `-s` parameter. Looking at logs it was a dead giveaway.

{% highlight tcsh %}
2016-02-27T22:15:33.956657+01:00 htpc nagios: wproc:   stderr line 01: sendmail: invalid option -- 's'
2016-02-27T22:15:33.956660+01:00 htpc nagios: wproc:   stderr line 02: sendmail: invalid option -- 's'
2016-02-27T22:15:33.956664+01:00 htpc nagios: wproc:   stderr line 03: sendmail: fatal: usage: sendmail [options]
{% endhighlight %}

In order to change `sendmail` to `mailx` for example we need to edit:

{% highlight tcsh %}
sudo nano /usr/local/nagios/etc/objects/commands.cfg
{% endhighlight %}

and change `notify-host-by-email` and `notify-service-by-email` commands:

{% highlight tcsh %}
# 'notify-host-by-email' command definition
define command{
	command_name	notify-host-by-email
	command_line	/usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /usr/bin/mailx -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
	}
	
# 'notify-service-by-email' command definition
define command{
	command_name	notify-service-by-email
	command_line	/usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /usr/bin/mailx -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
	}
{% endhighlight %}

Part before `| /usr/bin/mailx -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$` is just a print of the email body which is then piped into mail command.
Instead of the `/usr/bin/mailx` I had `/usr/sbin/sendmail` which doesn't accept `-s` parameter for subject. Changing this to `/usr/bin/mailx` enabled the email notifications to work.
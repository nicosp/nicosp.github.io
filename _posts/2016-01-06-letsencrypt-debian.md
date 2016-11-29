---
layout: post
title:  "Let's Encrypt Debian!"
date:   2016-01-06 02:39:00 +0200
categories: linux
---
Fire and forget setup for [letsencrypt][letsencrypt] in Debian (currently sid only is supported).
All commands must be run as root. Replace mydomain.com with your server hostname eg. www.example.com

# Installation
Install the letsencrypt client:

{% highlight shell %}
apt-get update
apt-get install certbot
{% endhighlight %}

Create a script to renew the certificate and reload affected services:

{% highlight shell %}
#!/bin/sh
certbot certonly -n --agree-tos --keep-until-expiring --webroot -w /var/www/html -d mydomain.com --renew-hook 'systemctl reload apache2 nginx dovecot postfix'
{% endhighlight %}

The script assumes you have a web server running on port 80 with webroot at /var/www/html. If you do not
have a web server on port 80 you must skip --webroot -w /var/www/html.

Add a cron job to run the script every day at 02:00.

{% highlight shell %}
0 2 * * * /usr/local/sbin/cert_update
{% endhighlight %}

# Configuration

## Apache
Add to the virtualhost configuration block:

{% highlight apache %}
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/mydomain.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/mydomain.com/privkey.pem
{% endhighlight %}

## nginx
Add to your server block for SSL:
{% highlight conf %}
ssl_certificate     /etc/letsencrypt/live/mydomain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;
{% endhighlight %}

## Dovecot
/etc/dovecot/conf.d/10-ssl.conf:
{% highlight conf %}
ssl = yes
ssl_cert = </etc/letsencrypt/live/mydomain.com/fullchain.pem
ssl_key = </etc/letsencrypt/live/mydomain.com/privkey.pem
{% endhighlight %}

## Postfix
/etc/postfix/main.cf:
{% highlight conf %}
smtpd_tls_cert_file=/etc/letsencrypt/live/mydomain.com/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/mydomain.com/privkey.pem
smtpd_use_tls=yes
{% endhighlight %}

[letsencrypt]: https://letsencrypt.org

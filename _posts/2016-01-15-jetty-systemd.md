---
layout: post
title:  "Jetty with systemd"
date:   2016-01-06 02:39:00 +0200
categories: linux
---

Create /etc/systemd/system/jetty.service

{% highlight shell %}
[Unit]
Description=Jetty Web Application Server
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/jetty/bin/jetty.sh start
ExecStop=/opt/jetty/bin/jetty.sh stop

[Install]
WantedBy=multi-user.target
{% endhighlight %}

{% highlight shell %}
systemctl enable jetty
service jetty start
{% endhighlight %}

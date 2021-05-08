---
layout: post
title:  "Install pgadmin4 on debian with nginx"
date:   2021-05-08 16:45:00 +0300
categories: linux
---
# Install required packages
As root:
{% highlight shell %}
echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" >/etc/apt/sources.list.d/pgadmin4.list
curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | apt-key add
apt update
apt install uwsgi-plugin-python3 pgadmin4-server
{% endhighlight %}

# Create user and paths
{% highlight shell %}
adduser --system --home /var/lib/pgadmin --disabled-login --shell /usr/sbin/nologin pgadmin4
mkdir /var/log/pgadmin
chown pgadmin4 /var/log/pgadmin
{% endhighlight %}

# Systemd unit file
Create /etc/systemd/system/pgadmin4.service
{% highlight INI %}
[Unit]
Description = PgAdmin4 uwsgi Service
After = network.target network-online.target
Wants = network-online.target

[Service]
User=pgadmin4
StateDirectory=pgadmin
RuntimeDirectory=pgadmin4
LogsDirectory=pgadmin
ExecStart=uwsgi \
      --socket /run/pgadmin4/pgadmin4.sock --chmod-socket=666 \
      --plugin python3 \
      -H /usr/pgadmin4/venv \
      --wsgi-file /usr/pgadmin4/web/pgAdmin4.wsgi \
      --processes 1 \
      --threads 25 \
      --chdir /usr/pgadmin4/web/ \
      --manage-script-name \
      --mount /pgadmin=pgAdmin4:app
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -INT $MAINPID
Restart=always
Type=notify
StandardError=syslog
NotifyAccess=all
KillSignal=SIGQUIT

[Install]
WantedBy = multi-user.target
{% endhighlight %}

# Configure pgadmin4
Create /etc/pgadmin/config_system.py

{% highlight python %}
LOG_FILE = '/var/log/pgadmin/pgadmin4.log'
SQLITE_PATH = '/var/lib/pgadmin/pgadmin4.db'
SESSION_DB_PATH = '/var/lib/pgadmin/sessions'
STORAGE_DIR = '/var/lib/pgadmin/storage'
{% endhighlight %}

# Configure nginx

{% highlight nginx %}
upstream pgadmin {
  server unix:///run/pgadmin4/pgadmin4.sock;
}

server {
...
  location ^~ /pgadmin {
    include uwsgi_params;
    uwsgi_pass  pgadmin;
  }
...
}

{% endhighlight %}

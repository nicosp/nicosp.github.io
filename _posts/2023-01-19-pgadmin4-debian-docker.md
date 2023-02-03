---
layout: post
title:  "Install pgadmin4 on debian with nginx+docker"
date:   2023-01-19 21:15:00 +0300
categories: linux
---
Based on the instructions at: https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html

# Configure nginx
{% highlight nginx %}
        location ^~ /pgadmin/ {
                include proxy_params;
                proxy_set_header X-Script-Name /pgadmin;
                proxy_set_header X-Scheme $scheme;
                proxy_pass http://localhost:5050;
                proxy_redirect off;
        }
{% endhighlight %}

# Create user and paths
{% highlight shell %}
adduser --system --home /var/lib/pgadmin --disabled-login --shell /usr/sbin/nologin pgadmin4
{% endhighlight %}

# Initial setup
As root:
{% highlight shell %}
docker run --name "pgadmin4" -e 'PGADMIN_LISTEN_PORT=5050' --net=host \
    -v /var/lib/pgadmin:/var/lib/pgadmin --user 117 \
    -e "PGADMIN_DEFAULT_EMAIL=user@domain.com" \
    -e "PGADMIN_DEFAULT_PASSWORD=SuperSecret" \dpage/pgadmin4
{% endhighlight %}

Go to http://localhost:5050 and login with user@domain.com / SuperSecret.
Create a user in the web interface with Administrator Role.

Make the docker container auto-start
{% highlight shell %}
docker container rm "pgadmin4"
docker run --name "pgadmin4"  -e 'PGADMIN_LISTEN_PORT=5050' --net=host -v /var/lib/pgadmin:/var/lib/pgadmin --user 117 --detach dpage/pgadmin4 --restart always
{% endhighlight %}

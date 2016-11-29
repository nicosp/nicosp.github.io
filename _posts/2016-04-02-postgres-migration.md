---
layout: post
title:  "Migrate Postgres to another server"
date:   2016-04-01 02:01:00 +0200
categories: linux
---
# Backup Original

{% highlight shell %}
su - postgres
pg_dumpall | gzip --best >/var/lib/postgresql/backup.sql.gz
{% endhighlight %}

# Restore
Replace 9.4 with the version of PostgresSQL you have installed.

{% highlight shell %}
scp root@ORIGINAL_IP:/var/lib/postgresql/backup.sql.gz /var/lib/postgresql
su - postgres
pg_dropcluster --stop 9.4 main
pg_createcluster --locale en_US.UTF-8  --start 9.4 main
zcat /var/lib/postgresq/backup.sql.gz | psql
{% endhighlight %}

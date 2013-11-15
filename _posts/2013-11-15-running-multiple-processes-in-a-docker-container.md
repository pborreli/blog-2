---
layout: post
title: Running multiple processes in a docker container
category: articles
tags: [docker]
comments: true
---

Starting with docker 0.6.5 you don't need pipework for that kind of scenario, you can use the new container linking feature. Basically, you tell docker to make a port from a container available to another container.

It's pretty easy to do, too.

What you want to do is have a container with php5-fpm (let's call this container `php5-fpm`) configured to listen on port 9000 and run it like so:

    docker run -d -p 9000 -name php php5-fpm /usr/sbin/php5-fpm -F

We run `php5-fpm` with the `-F` flag so that it does not daemonize. As you can see, we use `-name` to explicitely name our container. We will use this name to reference it in the link we are going to create with the nginx container.

Then you can run your nginx (called `nginx`) container:

    docker run -i -t -link php:php nginx /bin/bash

The `-link` option tells docker to link the `php` container under the alias `php`. The alias is mandatory.

We now have a shell in our nginx container, and we can retrieve the mapped ip and port of the `php5-fpm` container using the `env` command:

    root@061fe34bd07b:/# env
    HOSTNAME=061fe34bd07b
    TERM=xterm
    PHP_PORT=tcp://172.17.0.44:9000
    PHP_PORT_9000_TCP_PROTO=tcp
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    PWD=/etc/nginx/sites-enabled
    PHP_PORT_9000_TCP_PORT=9000
    SHLVL=1
    HOME=/
    PHP_PORT_9000_TCP=tcp://172.17.0.44:9000
    PHP_NAME=/crimson_squirrel9/php
    DEBIAN_FRONTEND=noninteractive
    PHP_PORT_9000_TCP_ADDR=172.17.0.44
    container=lxc
    OLDPWD=/
    _=/usr/bin/env

There are a number of interesting env vars here. The one we are looking for is `PHP_PORT`, since it gives the most complete information about the linked container:

    PHP_PORT=tcp://172.17.0.44:9000

You can now configure nginx's php5-fpm upstream to 172.17.0.44:9000, start it, and check that it works:

    /etc/init.d/nginx start
    curl http://127.0.0.1/index.php

Voila ! I skipped provisioning and configuration of containers since you seem to have got that right already ;)

Link to the official linking tutorials, using redis: http://docs.docker.io/en/latest/examples/linking_into_redis/
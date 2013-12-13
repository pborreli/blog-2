---
layout: post
title: Making docker containers communicate
category: articles
tags: [docker, networking]
comments: true
---

[Docker 0.6.5](http://blog.docker.io/2013/10/docker-0-6-5-links-container-naming-advanced-port-redirects-host-integration/) introduced a very interesting feature: container linking. Basically, you can tell docker to make a port from a container available to another container. And it's pretty easy to use, too. To demonstrate how container linking works, we will setup a quick php5-fpm + nginx configuration using one container for php-fpm and another for nginx.

This article assumes that you know how to build a container and that you already both a `php5-fpm` and `nginx` container. You can read more about building containers at [the official docker documentation](https://docs.docker.io/en/latest/use/builder/).

What you want to do is have the container with php5-fpm configured to listen on port 9000 and run it like so:

    $ docker run -d -p 9000 -name php55 php5-fpm /usr/sbin/php5-fpm -F

We run `php5-fpm` with the `-F` flag so that it does not daemonize. As you can see, we use `-name` to explicitly name our container instead of having to remember its commit id. I named it `php55`, that's for PHP 5.5, since you don't run an unsupported version of PHP do you?

We can now use this name to reference the php container in the link we are going to create with the nginx container:

    $ docker run -i -t -link php55:php nginx /bin/bash

The `-link` option tells docker to link the `php55` container under the alias `php`. The alias is mandatory and must not be empty (no cheating with `php55:` for example).

We now have a shell in our `nginx` container, and we can retrieve the mapped ip and port of the `php5-fpm` container using the `env` command:

    $ env | grep -E ^PHP
    PHP_PORT=tcp://172.17.0.44:9000
    PHP_PORT_9000_TCP_PROTO=tcp
    PHP_PORT_9000_TCP_PORT=9000
    PHP_PORT_9000_TCP=tcp://172.17.0.44:9000
    PHP_NAME=/crimson_squirrel9/php55
    PHP_PORT_9000_TCP_ADDR=172.17.0.44

There are a number of interesting environment vars docker exported for us. The one we are looking for is `PHP_PORT`, since it gives the most complete information about the linked container:

    PHP_PORT=tcp://172.17.0.44:9000

You can now configure nginx's php5-fpm upstream to `172.17.0.44:9000`, start it, and check that it works:

    /etc/init.d/nginx start
    curl http://127.0.0.1/index.php

Remember, your PHP source code has to be available on your php5-fpm container (maybe you uploaded it earlier, or ar using volumes to make it available to multiple containers).

> The official Docker documentation also has a very good [tutorial on containers linking](http://docs.docker.io/en/latest/examples/running_redis_service/), using redis, be sure to read it for more informations.
{:.note}
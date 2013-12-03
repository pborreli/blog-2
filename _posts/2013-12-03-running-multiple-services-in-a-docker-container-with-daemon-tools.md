---
layout: post
title: Running multiple services in a Docker container with daemontools
category: articles
tags: [docker, processes, daemontools]
comments: true
published: true
---

One concept people often overlook about Docker is that there can only be one process running at a time in Docker, and when this process dies, the container stops. What that means is you can not simply start daemons in your container using the usual `/etc/init.d/nginx start` for example, because while nginx will effectively start, it's going to start in the background and your primary process will exit, stopping the container by the same occasion.

So how do you get multiple services to run in a Docker container? There are several ways, but it helps to see your initial process as an [`init`](http://en.wikipedia.org/wiki/Init) process since after all, that is what it is. By leveraging this concept and using a process manager, we can run all our services and make sure they keep running, restarting them if necessary.

To make things simpler, we are going to use Docker's Ubuntu base image, more precisely, the `precise` one (pun intended, sorry for that), so let's pull it from the public index and open up a shell in a new container:

    $ docker pull ubuntu:precise
    $ docker run -i -t run ubuntu:precise /bin/bash


> Everything should work exactly the same with more recent versions of ubuntu, but `precise` is the version I tested, so that will be the one I talk about.
{:.note}

## Installing and configuring services

First things first, install OpenSSH:

    $ apt-get install -y openssh-server

We are going to use [daemontools](http://cr.yp.to/daemontools.html) in this article. The great thing about daemontools is that it follows the UNIX tooling philosophy by not doing too much at once. Instead, it provides a set of tools that can be used together to achieve great control on how things run on your system. There is a few other process managers to chose from and you could use anything from `supervisord` to `monit`, but debating the pros and cons of them is beyond the scope of this tutorial.

There is an ubuntu package for `daemontools`, but it's in the `universe` package repository, so you will need to add it and update apt-get first:

    $ echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
    $ apt-get update

You can now install `daemontools`:

    $ apt-get install -y daemontools

Daemontools expects services to be defined each in their own directory. Each service directory must contain an executable file named `run`. This executable can be anything that actually is executable, a binary compiled executable, a shell script, a ruby/python/whatever executable, you name it. The only requirement is that it is executable (using `chmod +x`).

We will store our services definitions in `/etc/services`, so create that directory if it does not already exists...

    $ mkdir /etc/services

...and since we want to monitor an sshd process, create an `sshd` directory in it...

    $ mkdir /etc/services/sshd

...and finally create a `run` script in this directory with this content:

    #!/bin/bash
    exec /usr/bin/sshd

> You might want to install an editor to create the file, or you could just as well use bash's heredoc syntax with `cat`:
>
>     $ cat > /etc/services/sshd/run <<EOF
>     > #!/bin/bash
>     > exec /usr/sbin/sshd
>     > EOF
{:.note}

The first line, `#!/bin/bash` is called a __shebang__ and tells the system what interpreter to use when executing this file. `exec` is a bash builtin that replaces the current shell with the supplied command, which will crowding our processes list with unnecessary bash processes.

Do not forget to make this script executable:

    $ chmod +x /etc/services/sshd/run

We are now ready to run `svscan`, which is `daemontools`' utility to scan a services directory and start processes:

    $ svscan /etc/services

You should have the following output:

    Missing privilege separation directory: /var/run/sshd

As the message states, sshd requires a `/var/run/sshd` for something related to privilege separation. That seems important, so let's stop `svscan` if not already done (you can hit `^C`) and create the directory:

    $ mkdir /var/run/sshd

You can now restart `svscan` and admire the lack of any output, which means that everything works fine!

# Making a reusable image out of it

It's all well and good that we are able to run an sshd inside our container, but as soon as we will exit the container all this work will be lost. Fortunately, Docker has a couple options for us to persist this work.

First, we can commit the container and obtain a brand new reusable image out of it. The prompt in your container contained your container's id. For example, if your prompt was something like `root@44c1ee41ae70`, the container id is `44c1ee41ae70`. Another way of finding the id of a stopped container is `docker ps -a` (the `-a` option stands for `all`), used in conjunction with the `-n` option to limit results:

    $ docker ps -a -n 1
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    44c1ee41ae70        ubuntu:precise      /bin/bash           9 minutes ago       Exit 130                                agitated_torvalds

Commit this container to a new `ssh` image by using `docker commit`:

    $ docker commit 44c1ee41ae70 ssh

You can now reuse your newly created image with `docker run`:

    $ docker run -i -t ssh /usr/bin/svscan /etc/services

And check that everything works fine from another terminal:

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
    4aa184718760        ssh:latest          /usr/bin/svscan /etc   16 seconds ago      Up 16 seconds                           high_mclean
    $ docker top 4aa184718760
    PID                 TTY                 TIME                CMD
    11264               pts/9               00:00:00            svscan
    11309               pts/9               00:00:00            supervise
    11311               ?                   00:00:00            sshd

# Automating things with a Dockerfile

Cherry on the cake, you can easily automate creation of such images with the use of a Dockerfile. [A Dockerfile](http://docs.docker.io/en/latest/use/builder/) is a Docker configuration file that tells how to build an image in a predictable and repeatable way. Our Dockerfile will look like that:

    FROM ubuntu:precise

    RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
    RUN apt-get update
    RUN apt-get install -y openssh-server daemontools
    RUN mkdir -p /etc/services/sshd /var/run/sshd
    RUN echo "#!/bin/bash\nexec /usr/sbin/sshd" > /etc/services/sshd/run
    RUN chmod +x /etc/services/sshd/run

    EXPOSE 22

    ENTRYPOINT /usr/bin/svscan /etc/services/

The `ENTRYPOINT` directive allows to define a command to be run when the container is created, and the `EXPOSE` tells Docker that we will want access to tcp port 22 of this container. Put all that in a Dockerfile and run `docker build`:

    docker build - ssh < Dockerfile

And you can now run your container like that:

    docker run -P ssh

The `-P` option tells Docker to publish all `EXPOSE`d ports, so we get a nicely mapped port to ssh to that we can retrieve using `docker port`:

    docker port e56733a0ceaf 22

Where `e56733a0ceaf`, of course, is the id of your newly created container.
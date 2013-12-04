---
layout: post
title: Managing permissions in your Docker volumes
category: articles
tags: [docker, vagrant, shared volumes]
comments: true
published: true
---

We saw yesterday [how to share a directory to Docker through Vagrant]({{ site.baseurl }}{% post_url 2013-12-03-sharing-a-directory-to-docker-through-vagrant %}) and this raised at list one question: how to manage permissions inside it. A friend of mine wants to store some database files inside the shared volume (so that they can restart the VM without losing database changes, fair enough). In order to achieve that, the database user (`postgre` in this case) has to own the database directory (we will call it `datadir`).

> Since there will be commands both in the Docker container and host, command prompts in this article will respectively read `docker $` and `host $` for clarity.
{:.note}

The problem we have here is a problem of UIDs. Let's take a look at what happens by default (with an OSX host):

    docker $ ls -l /var/www
    total 4
    drwxr-xr-x 2 501 dialout 68 Dec  4  2013 datadir

As you can see, the `datadir` directory belongs to UID `501` and to the `dialout` group. This can vary on your system depending on the UID of your user, but the principles stay the same. What exactly is this `dialout` group you ask? [According to the internet](http://www.togaware.com/linux/survivor/Standard_Groups.html) (and you know the internet is always right), it is a standard linux group used to give users access to __ppp and isdn devices__. What does it have to do with us? Not much. To get a better understand, we're going to use the `-n` option of the `ls` command:

    docker $ ls -ln /var/www
    total 4
    drwxr-xr-x 2 501 20 68 Dec  4  2013 datadir

What do we learn here? `datadir` actually belongs to GID `20`. So where are these UIDs and GIDs coming from? There is only one place they can come from: the host system. Both NFS and Docker preserve UIDs and GIDs, so what you see in Docker are actually the numbers used by the OSX host. We can check that easily by running `ls -ln` and `id` on the host:

    host $ ls -ln
    total 0
    drwxr-xr-x  2 501  20  68  4 Dec 10:41 datadir
    host $ id
    uid=501(geoffrey) gid=20(staff)

As you can see, `501` is the UID of my user, and `20` is OSX's default `staff` group's GID.

There are now two possibilities. The simplest is to force the UID of the `postgre` user in the container:

    docker $ useradd --uid 501 postgre
    docker $ ls -l /var/www
    total 4
    drwxr-xr-x 2 postgre dialout 68 Dec  4  2013 datadir

Now the `postgre` user can do anything in the `datadir` directory, but also in the whole shared volume, since all the other files also belong to UID `501`.

The other possibility is to chose an UID that is available both on the host, the VM and the Docker host (UIDs around `2000` should be more than ok) and change ownership of the `datadir` directory to that UID on the host:

    host $ sudo chown 2000 datadir/
    Password:
    host $ ls -l
    total 0
    drwxr-xr-x  2 2000  staff  68  4 Dec 10:41 datadir

You can now create a `postgre` user with UID `2000` in the container that will only own the `datadir` directory!
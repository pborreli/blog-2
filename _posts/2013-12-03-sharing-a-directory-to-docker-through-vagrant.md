---
layout: post
title: Sharing a directory to Docker through Vagrant
category: articles
tags: [docker, vagrant]
comments: true
published: true
---

That's a question I've been asked a lot lately, although that's fairly easy to do when you know exactly what you want to do.

Here's the setup: you run Docker through [the official Vagrant image](http://docs.docker.io/en/latest/installation/vagrant/), and want to share a directory on your computer to the Docker containers running in the VM. You don't have many choices. Volumes have to be available on Docker's host, so that means you first have to share directories to the Vagrant VM, using the `synced_folder` directive (we will assume files on your computer are in `/Projects/AwesomeProject` and you want to share them to Docker in `/var/www`):

    config.vm.synced_folders "/Projects/AwesomeProject", "/vagrant", :nfs => true

In this example, we share the directory to `/vagrant` since it's the most commonly used example directory for `synced_folders`, but it could be any directory really. We also add the `:nfs => true` parameter to tell Vagrant we want to share files using NFS rather than the default vboxfs, since it's known to be more efficient with large numbers of files. Please note that Vagrant __does not__ support NFS on Windows hosts.

Now fire up your VM (`vagrant up`) and once it's booted, check that `/vagrant` contains your files. You can now attach this directory as a volume to any Docker container, using the `-v` option of the `docker run` command:

    docker run -i -t -v /vagrant:/var/www ubuntu /bin/bash

You should get a shell inside the container, and be able to check that your files are present in `/var/www`, and voila! You're all set.
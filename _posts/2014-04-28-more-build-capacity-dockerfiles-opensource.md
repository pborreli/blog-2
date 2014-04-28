---
layout: post
title: Adding more build capacity, Dockerfiles and OpenSource
tags: [stage1, capacity, dockerfiles, opensource]
category: stage1
comments: true
---

__TL;DR__: Stage1 can now easily add capacity, supports [Dockerfiles](http://help.stage1.io/article/using-my-own-dockerfile/) and is officially __free for OpenSource__. [Register now for the private beta](http://stage1.io).

It's been a long time since the last blog post, but we have a good excuse: we were busy building your next staging platform. During the last few months, we worked hard to implement a few requested features, and to prepare for the future of Stage1: OpenSource. Being ourselves huge fan of OpenSource, we wanted to provide Stage1 for OpenSource projects, for free.

To fully support OpenSource projects, we had two challenges to tackle: being able to add capacity, and supporting arbitrary (web) technologies.

The challenge with capacity — compared to similar services — is that we keep a build artifact. And by build artifact, I don't mean we have a few thousand lines of logs in our database. I mean we keep a running instance of your app. Or two. Or more actually. Resource-wise, it's not really the same deal. In order to handle the load we needed to be able to add more capacity in an easy way. Easy like just throwing more servers. That's just what we did, and we can now add an arbitrary number of builder behind Stage1, as easily as pluging the server to our existing infrastucture.

With that solved, we were left with only one entry barrier for OpenSource projects of the world: supporting them. Stage1 was designed from the ground up to support [Symfony2](http://symfony.com/) projects (for various reasons, but mostly because you have to start somewhere), and other projects based on other technologies like [Laravel](http://laravel.com/), [Silex](http://silex.sensiolabs.org/), or even [Rails](http://rubyonrails.org/), [Django](https://www.djangoproject.com/), etc. could not leverage the power of Stage1.

Being based on [Docker](http://docker.io), the obvious solution was to leverage Dockerfiles, and that is just what we did. Starting today, all you have to do is include a Dockerfile at the root of your project and Stage1 will automatically detect it and use it to build your staging instance. It's that easy. You can read more about [Stage1 support for Dockerfiles on our support site](http://help.stage1.io/article/using-my-own-dockerfile/).

So with both these problems solved, I am delighted to announce that __Stage1 is now officially free for OpenSource projects, forever__. And if you are a company willing to sponsorize capacity for OpenSource builds, [we'd love to talk to you](mailto:geoffrey@stage1.io?subject=OSS)!

And since we're such big fans of OpenSource, we are also going to commit ourselves to opensource most of the components that make Stage1. We already started with a few of them, there's our [Docker-PHP client](https://github.com/stage1/docker-php), [Aldis](https://github.com/stage1/aldis), which routes containers' output through our message distribution system, and [Yuhao](https://github.com/stage1/yuhao) which is at the very heart of our builder, but there's more coming! And eventually, our goal is to make our whole codebase open.

The service is still in beta, but we're going to start accepting beta signups faster than before, so than more people can enjoy the awesomeness of automatic and interactive staging, so do not hesitate and [register now](http://stage1.io) for the private beta!
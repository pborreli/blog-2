---
layout: post
title: Staging as a communication tool
publish: true
tags: staging, communication
category: stage1
comments: true
---

A __staging platform__ is a vague notion. Before starting to communicate about **Stage1**, I conducted a little survey asking people how they did staging, and what they thought the ideal staging platform would be. Not only did I get an overwhelming lot of answers, but the diversity of them staggered me, too. From a step in the developer's workflow to something used to debug production, the range of definitions covered areas I didn't even think of. That's when I understood that Stage1, as a product, would only fit very few of those definitions and that I had to set a clear path for it.

Analyzing the survey's result, three concepts emerged:

1. Something that helps understand what's going on in production
2. Something that helps validating a project
3. Something that helps communicating with the client

The first proposition, __something that helps understand what's going on in production__, is interesting. It's something one could call __reactive staging__. This is not something that occurs before production, but after (or, if you wish, at the same time). It is most often used to debug hard to reproduce bugs, things that you need production data to work on for example. It requires precise duplication of the production environment, sometimes to the point of running on the same networks. This is obviously helpful, and I would have killed to have something like this during my agency days. Push a button, get an exact replica of the prod that I can toy with and break. But that is not staging as I define it.

__Something that helps validating a project__ is nice. It's called recetting. And it's used mostly by people still stuck in a waterfall project management style. Did I say nice? Sorry, I meant useless.

__Something that helps communicating with the client__? Now we're talking. A lot of people asked for this, just not in these exact terms. The most common way to express it is __I need to show progress to my client and get feedback__. This is communication. You need to **communicate** progress to your client. That's what Stage1 is meant to be: a communication platform.

There are a number of ways to communicate with your client. You could communicate by email or IM, writing down bulletpoint lists of features, trying to describe them the best you can. You could have a sharescreen session to demonstrate progress on the project. You could meet in person with the client so he can actually get to use the product. But would it be any good? You can't show off an interface by email. You can't really get the feel of the product by just watching someone using it. And having the client move to your place is not always convenient. Those are not good way of communicating.

Most agile methodologies advocate some sort of __demo day__ at the end of each sprint so they can show-off progress and allow clients to play with the product. This is good, but is it enough? Most team I've worked in had two weeks sprints, which mean the client has to wait two weeks before having a progress report. Sometimes they get information during the sprint, but that's it. Two weeks. That's quite some time.

Wouldn't it be better if the client could play with new features as soon as they reach the repository? This way, he can see right away what works and what doesn't, send feedback very quickly, and the development team can react accordingly. __Every day is demo day__. The feedback loop is shortened and strengthened at the same time.

That is what Stage1 aims to be. __A platform that helps communicate__ project progress by enabling your client to use the project at each and every stage of its development, and easing feedback gathering at the same time. In an upcoming series of posts, I will tell you about how we're going to do that, so stay tuned!
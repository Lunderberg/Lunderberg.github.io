---
layout: post
title: Github.io, Comments
tags: [site-info]
date: 2014-10-14 22:00:00
---

As far as I have used it, jekyll is a great platform,
  especially with the ease of publishing on github.io pages.
However, it is limited to static pages, which makes it difficult to have community interaction.
I can make new posts from any browser window, but readers cannot easily leave comments.

One workaround is to use [github issues as comment threads](http://ivanzuzak.info/2011/02/18/github-hosted-comments-for-github-hosted-blogs.html).
The upside is that all comment threads are connected to the repository, and can be easily located.
The downside is that leaving comments requires a github account.
If I were writing solely about programming, it could be assumed that most readers will have a github account anyways.
As it is, I would rather have more flexibility.

It seems that the standard way to have comment fields in a jekyll blog is by using disqus.
Disqus stores the comments on their servers, which are retrieved using javascript.
The jekyll page needs only to link to the appropriate javascript file.
This allows for a wider range of sign-ins, and does not need manual setup for each post.
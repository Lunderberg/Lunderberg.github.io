---
layout: post
title: Tags in Jekyll
tags: [site-info]
---

If I knew what I was doing, I wouldn't be taking so long in trivial tasks.
But, then again, if I knew what I was doing beforehand, I wouldn't be having anywhere near as much fun.
After all, it's a learning experience.
Anyways, I know that I can find old posts better if I can apply categories,
  and I never really liked the single-category-per-item mindset that is necessary for folders to work.
Therefore, setting up a framework for using tags.

I am using a setup similar to that given in [minddust.com's](http://minddust.com/post/tags-and-categories-on-github-pages) example.
The advantage is that I do not need to compile the jekyll site locally, nor do I need to install any additional plugins.
The downside is that I need to make modifications whenever I add a new type of tag.
On the bright side, there are only two changes that need to be made.

1. Adding an entry in `_data/tags.yml` for the new tag.
2. Adding an empty template in `blog/tags` for the new tag.

When adding a new post, any tags listed in the front-matter block will be added to the appropriate index.
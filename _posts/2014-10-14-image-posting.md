---
layout: post
title: Github.io Image Hosting
tags: [site-info, omnicolor-images]
---

Adding images in a post can be done with just a simple image tag,
  `![like this](http://image.url/put/here)`.
This leaves only the question of where to host the images.

One option is to make an issue on the repository and post images in that thread.
Those images have some public URL, which can then be linked against.
However, these images would then have a URL that is not associated with the blog itself,
which could become confusing.

Another option would be to add the images to the repository.
This has the advantage of sane URLs.
In addition, the site can be replicated locally by cloning the repository,
  without external dependencies.

However, this then requires disk space for all of the assets,
which might not be desired, especially for quick edits.
As a compromise, the assets could be placed into a submodule.
This submodule could be cloned only when desired,
allowing for both local reproduction of the site, and low bandwidth edits.
This is the method that I will be using for this blog.

As a taste of what is to come, below is an image from a program that I have been working on.

![omnicolor image](/assets/omnicolor-images/512x512_uniform-rgb_frontier-growth.png)
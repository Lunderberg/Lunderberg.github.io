---
layout: post
title: The Importance of Algorithms
tags: [omnicolor-images,c++]
---

When making the omnicolor images, my reference point has been the time necessary to make a 1920x1080 image.
The first implementation took about five hours.
By the end, the same size image took twenty seconds to generate.

By profiling, one particular function took the vast majority of the runtime.
Recall from the previous post that the images are generated by repeatedly choose a pixel,
then selecting the closest unused color to the neighbors of the pixel.
It is this selection that takes the longest amount of time.

My initial implementation was a naive linear search.
I had two million colors, one of which must be the closest.
Therefore, I calculated the distance from my desired color to each color in the palette.
The color with the shortest distance was used.

This worked, and was able to produce very nice-looking images.
However, it took quite a bit of computer time to do so,
especially as I scaled to larger images.
The image below is the first wallpaper sized image that I made,
taking about 5 hours for the program to run.

![first_wallpaper](/assets/omnicolor-images/first_1920x1080.png)

Now, for a one-time program, 5 hours of runtime isn't that bad.
But I wanted to be able to tweak parameters, then see the results,
  then tweak again.
With that in mind, 5 hours would be horrendous.
Therefore, optimizing the code.

When working on optimizations, I found it helpful to think geometrically.
A color has three values, for the red, blue, and green brightnesses.
These can be visualized as x, y, and z coordinates.

If I had one-dimensional points,
  I would use some binary search tree to find the closest point.
However, this assumes that the search space is well-ordered.
In one dimension, the concept of "greater than" means something.
In more than one dimension, it no longer does,
  as, for example, neither (3,6) nor (4,5) is greater than the other.

There is still some structure that can be exploited.
While points cannot be greater or less than each other,
  their magnitudes can.
If I keep the colors sorted according to magnitude,
I can start my search at points located close to the desired magnitude.

A diagram of this method is shown below, for two dimensions.
The blue points are the existing color palette,
  and the red point is the desired point.
The gray band is the region that has been search so far.

![magnitude-search](/assets/omnicolor-images/magnitude-search.png)

All points outside the gray band must be at least as far as the thickness of the band.
Therefore, if I find a blue point that is closer to the red point than the thickness of the band,
  I can avoid checking any points outside the band,
  since they are guaranteed to be farther away.
I still need to check against every point within the band,
  but that is a drastic reduction from checking against every point.

This reduced my search space from three dimensions down to two,
since I was effectively searching along the boundary of a sphere.
The complexity was effectively reduced from $\mathcal{O}(n)$
to $\mathcal{O}(n^{2/3})$ in the best case scenario.
This improved the runtime of the program from 5 hours to about 20 minutes.

This was much better, but still not as fast as I would like.
A google search ended up finding that this was a
[common problem](http://en.wikipedia.org/wiki/Nearest_neighbor_search),
with some standard solutions.
The most common solution was to build a [kd-tree](http://en.wikipedia.org/wiki/K-d_tree), which has $\mathcal{O}(\log n)$.

After implementing a kd-tree, the run times were reduced to about 90 seconds.
At this point, it became a matter of small optimizations,
which are summarized in the table below.

| Time   | Change                                                      |
|--------|-------------------------------------------------------------|
| 93 sec | Base                                                        |
| 71 sec | Replacing `std::shared_ptr` with `std::unique_ptr`, C-style |
| 27 sec | Doing a linear search after reducing to 50 points           |
| 22 sec | Avoiding sqrt by tracking dist^2 instead of dist            |

The `shared_ptr`s were originally used for ease of implementation,
  and were slowly taken out entirely.
They allow for more flexibility, but have additional costs for runtime checks.
Wherever appropriate, I replaced them with `unique_ptr`s,
  occasionally using C-style pointers for internal checks.

The most dramatic improvement came when changing the tree structure.
Initially, I recursively split space until there was only one point possible.
This was changed so that space was split until there were up to fifty points in the possible region.
These points were then iterated through manually to find the closest.
This does require more calculations, but it offers memory access improvements.
This way, fewer lookups are needed, as the entire end node can be loaded into the CPU cache.

Throughout this process, the algorithm performed the same task,
just in a smarter manner.
The qualitative feel of the images stayed the same,
as evidenced by the new image below.

![omnicolor-wallpaper](/assets/omnicolor-images/1920x1080_uniform-rgb_frontier-growth.png)

With a runtime of 22 seconds, it is now fast enough to use interactively.
In the future, I intend to have the algorithm show its progress to the user
  as it goes, not just an image or a video output.

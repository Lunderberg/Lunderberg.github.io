---
layout: post
title: Omnicolor Images
tags: [omnicolor-images]
---

Given a color palette,
  make an image that uses each color exactly once.

![omnicolor image](/assets/omnicolor-images/512x512_uniform-rgb_frontier-growth_2.png)

Obviously, this is not done by placing the pixels in order, as that would make a rather boring image.
Nor am I doing so by hand, because
Programmatically, for each pixel, I choose the color that is closest to its neighbors.

In psuedo-code, the algorithm is as follows.

```
frontier = [random_point]
palette = color_palette()
while has_points(frontier){
  new_point = choose_location(frontier)
  desired_color = average_neighbor_color(new_point)
  new_color = find_closest(palette, desired_color)

  frontier.remove(new_point)
  palette.remove(new_color)
  image.set(new_point, new_color)

  frontier.grow_around(new_point)
}
```

![omnicolor image](/assets/omnicolor-images/512x512_uniform-rgb_frontier-growth_3.png)

Since the algorithm proceeds point by point, one can show the progression as it places pixels.
The video below shows how the algorithm proceeds.

<iframe width="600" height="400" src="http://youtube.com/embed/Czkt_LxgoZs" frameborder="0" allowfullscreen></iframe>

By adjusting the `choose_location()` function, I am able to produce a range of effects, such as the patchwork filling in the above video.

This project was inspired by this [stackexchange challenge](http://codegolf.stackexchange.com/questions/22144/images-with-all-colors).

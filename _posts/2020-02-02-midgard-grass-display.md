---
layout: post
title: Midgard, Grass Display
tags: [c++, midgard]
---

Now that the [grass can grow on the backend]({% post_url
2020-01-29-midgard-growing-grass-2 %}), it needs to be displayed on
the browser-based front-end.  This should be fairly straightforward,
as the implementation was designed with ease of display in mind.

Each bitfield of grass defines a region, with smaller regions being
used where further detail is needed.  These smaller regions overlay on
top of the previous ones, overriding their values.  For displaying the
full field, we can start by drawing the largest field, then draw any
subfields overtop it.  Since the fields on the backend are ordered
according to a depth-first search, this same ordering works for
drawing.

A dummy canvas is generated, and used to make an 8x8 image.  This 8x8
image is then scaled to the region that it covers, and drawn.

![grass growing](/assets/midgard/2020-02-02_grass-growing_regions-visible.gif)

And it works, though with some visual artifacts.  The larger regions
are visible, where they shouldn't be, and individual tiles have blurry
edges.  This is an artifact of upscaling the 8x8 images.  In my
browser, this defaults to using a bilinear interpolation, when I want
to use nearest neighbor instead.  To change this, I need to set
`ctx.imageSmoothingEnabled = false;`

![grass growing](/assets/midgard/2020-02-02_grass-growing_nearest-neighbor.gif)

Lastly, the coordinate system.  My background is primarily in
math/physics, and so I tend to think of increasing the y coordinate as
being "up".  In 2D computer graphics, y is usually "down", being
measured from the top of the screen.  Being foolish, I like to adapt
the world to myself, and therefore want to correct this.

![grass growing](/assets/midgard/2020-02-02_grass-growing_correct-y.gif)
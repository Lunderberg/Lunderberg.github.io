---
layout: post
title: Midgard, Growing Grass
tags: [c++, midgard]
---

Now that I have an efficient way to represent a grassy field, I need
to allow it to grow.  In each iteration, grass will grow from grassy
tiles to any barren tiles surrounding the grassy tile.

At the lowest-level grid, this can be specified using bitwise
operations.

```c++
Bitfield no_left_side = 0x7f7f7f7f7f7f7f7f;
Bitfield no_right_side = 0xfefefefefefefefe;
Bitfield new_field = field |
                     ((field & no_left_side) << 1) |
                     ((field & no_right_side) >> 1) |
                     (field << 8) |
                     (field >> 8);
```

Bitshifting by 1 moves the pattern left/right by one tile, and
bitshifting by 8 moves the pattern up/down by one tile.  When moving
left or right, we need to zero-out bits on one side to prevent moving
over the grid's edge.

In addition, each lowest-level grid needs to be able to be influenced
by the grids next to it.  Each grid needs to look in the adjacent
tiles, to see whether grass should grow into it or not.  This needs to
be done as a pre-step, before any grass grows, to prevent accidental
growth by two tiles in a single iteration.

It is here that I am running into issues.  The lowest-level grid can
easily look at the adjacent grids.  However, suppose there isn't a
lowest-level grid, because it has been entirely filled in.  In that
case, the lowest-level grids need to be generated, in order to have
that level of detail be expressed.

However, it is not sufficient to see grass adjacent to barren ground
in a higher-level grid, because either of those tiles might be
overridden in a lower-level grid.  The next step is to allow
recursion, declaring a lowest-level grid to be necessary only if that
tile in the current grid has no overrides, and otherwise recursing
down to that lower level of detail.  The issue with that approach is
that finer detail may be needed not because the current grid has
overrides, but because an adjacent grid has overrides that need to
grow into the current grid.

It is here that I am getting stuck.  I feel that there must be a clean
algorithm that is amenable to this data structure, but I will need
more thought in order to find it.

---
layout: post
title: Midgard, Growing Grass, Part 2
tags: [c++, midgard]
---

In the [previous post]({% post_url 2020-01-21-midgard-growing-grass
%}), I attempted to find an efficient algorithm to have grass grow
along a field of bits, where the bitfield is [defined recursively]({%
post_url 2020-01-20-midgard-recursive-bitfield %}).  I think I have
managed to figure it out.

The grass growing occurs in two phases.  First, all new growth is
found.  Second, the new growth is applied to the bitfields, and any
completely full or completely empty bitfields are removed.

The first step is the hardest, because the data structure does not
have many constraints.  A bitfield's value does not necessarily define
the value of any tiles at all, because it may be overridden by a
smaller grid.  Or, it may be overridden, but only partially.

The key was processing each tile recursively.  If a subfield exists,
then that subfield needs to be recursed into.  If a subfield's
neighbor exists, then it still needs to be recursed into, in order to
handle detail at that level.  If a subfield doesn't exist, and if none
of its neighbors exist, then the border is entirely defined by the
values stored at the current level, and required changes can be
determined without any additional recursion.

This is aided by the addressing scheme of bitfields.  The bitfield
address contains the location in the top-most bitfield as its highest
bits, then the next level down, then the next.  The layer is stored in
the bottom 4 bits, but in reverse order.  This way, iterating through
the addresses in order performs a depth-first search of the bitfields.
Whether a bitfield or any of its subfields exist can then be
determined in $\mathcal{O}(\log n)$ time.

![grass growing](/assets/midgard/2020-01-28_grass-growing.gif)

The lines in this animation show the divisions between the different
8x8 grids making up a large 64x64 grid.  The edges are also defined to
wrap around each other in this instance.
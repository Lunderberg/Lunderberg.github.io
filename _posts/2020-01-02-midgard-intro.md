---
layout: post
title: Simulated World
tags: [c++, midgard]
---

A few years ago, I worked on a C++ implementation of
[NEAT](https://en.wikipedia.org/wiki/Neuroevolution_of_augmenting_topologies),
a method for generating novel neural net topologies.  Rather than
having a fixed topology, varying only the weights of a neural net,
NEAT also varies which nodes are connected to each other.  This allows
for creation of minimal structures, capable of solving the problem
presented without having significant excess neurons.

The implementation did well when tested against standard benchmarks,
such as balancing of an inverted pendulum.  However, I wanted to test
it on a more open-ended situation as well.  To that end, I am making a
simple simulated world, in which agents controlled by neural nets will
be able to interact.  This has been done before, but will be an
enjoyable project overall.

The main program will be designed in C++, to interface with my
existing NEAT library.  The user interface will be displayed over the
web, as it leads to easier demonstrations.  WebSockets will be used to
communicate between the C++ backend and the browser.

Before any agents are added to the simulation, I want to have some
structure to the world they will inhabit.  The agents will be able to
move around and eat, so they will need something to eat.  Therefore,
grass will grow, starting from an initial seeding of grass.  Agents
will need to move in order to avoid overharvesting their living area.

For a first implementation, each tile has some amount of grass on it,
from 0 to 1.  After each time step, the amount of grass in a tile is
averaged between itself and its neighbors.

| Iteration | Food Distribution |
|-----------|-------------------|
| 0  | {::nomarkdown}<img src="/assets/midgard/2020-01-02_iteration-0.png" width=100>{:/} |
| 1  | {::nomarkdown}<img src="/assets/midgard/2020-01-02_iteration-1.png" width=100>{:/} |
| 2  | {::nomarkdown}<img src="/assets/midgard/2020-01-02_iteration-2.png" width=100>{:/} |

Grass spreads, but never grows.  The total amount of grass in the map
stays the same in this implementation.  It is good for testing the
communication between browser and engine, but will need to be improved
on later.

Project page: [Midgard](https://github.com/Lunderberg/midgard)

---
layout: post
title: Simulated World, Threading
tags: [c++, midgard]
---

Currently, everything that happens on the C++ backend for Midgard is
on a single thread.  This includes the static file serving, the
websocket handling, and the simulation itself all occur on a single
thread.  All computation is done at request from the browser, and
can't continue indefinitely, because the thread needs to return to
waiting for the next thread.

![One click per iter](/assets/midgard/2020-01-05_single-iter-per-click.gif)

Overall, it works, but the browser needs to request each successive
step of the simulation.  While the browser could request each new
computation automatically, this would be problematic if multiple
clients connect at the same time.

The C++ backend consists of three main classes.  First, there is a
WebServer, which handles receiving the connections themselves.
Second, there is a WorldSim, which handles the simulation of the
world.  Finally, there is a WorldController, which connects the two.
It interprets the JSON-encoded commands from the browser, makes calls
into the WorldSim, and encodes the response to be sent back to the
browser.  This WorldController needs to own a separate thread, on
which the WorldSim itself runs.

For each request, it is pushed onto a worker queue.  The worker thread
reads the request and parses it.  Several iterations of the simulation
can then be run, with the worker thread passing back status messages
as it goes.  This is show below, with a one second sleep added to make
the changes visible.

![Several click per iter](/assets/midgard/2020-01-08_several-iter-per-click.gif)

There are several steps for improvement that will can be made later.
Currently, the worker thread reads each message sequentially.
Instead, a message to stop iterating or to reset the world should
interrupt the current calculation.
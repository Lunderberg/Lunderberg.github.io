---
layout: post
title: C++ Makefile, Command-Line Variables
tags: [c++, make]
---

As of the end of the [previous post]({% post_url 2015-08-10-cpp-makefile-library %}),
  we have a working makefile.
Now, I want to start following better practices,
  so that it can be more flexible for interactive use.

Makefile variables can be specified on the command line.
When they are passed, they override variables specified within the makefile.
In this way, behavior can be changed without needing to change the makefile.
For example, if you want to enable profiling with gprof,
  you can add the `-pg` flag to compilation and linking immediately.

Currently, our makefiles use some non-standard variables.
We are using `CFLAGS` for variables passed to the C++ compiler,
  and `LFLAGS` for variables passed to the C++ linker.
The standard variables are specified [here](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html).
The rules that we are interested in are as follows.

* `CXX`: The C++ compiler to use.
* `CPPFLAGS`: Flags intended for the C pre-processor.
              This includes any `-D`, `-U`, or `-I` flags
* `CXXFLAGS`: Flags intended for the C++ compiler.
              For example, `-std=c++11`.
* `LDFLAGS`: Flags intended for the linker.
             This includes any `-L` commands.
* `LDLIBS`: Flags specifying libraries to be linked against.
            This should only include `-l` commands.

Next, we want to edit our build rules such that they use these new variables.
We change the compilation rules to use the new variables.

```make
bin/%: build/%.o $(O_FILES)
	mkdir -p $(@D)
	$(CXX) $(LDFLAGS) $^ $(LDLIBS) -o $@

build/%.o: %.cc
	mkdir -p $(@D)
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

We now need to make a distinction between optional arguments,
  which someone might want to change at any moment,
  and mandatory arguments, which are needed to compile.
The level of optimization, for example, is optional.
Which folders contain include files, on the other hand, is not.
We want to give the user flexibility without forcing them
  to remember each argument necessary for compiling.
Therefore, we will make use of the
  [override directive](https://www.gnu.org/software/make/manual/html_node/Override-Directive.html)

```make
CXX      = g++
CPPFLAGS =
CXXFLAGS = -g -O3
LDFLAGS  =
LDLIBS   =

override CPPFLAGS += -Iinclude
```

Here, the initial value of `CPPFLAGS` can be set by the user on the command line.
Whatever is given will have `-Iinclude` added to it,
  without the user needing to add it on the command line.
The `-g` and `-O3` in `CXXFLAGS`, on the other hand,
  are not necessary for correct compilation, and can be omitted without error.

Now, I could enable profiling by running the command

```
make clean && make CXXFLAGS=-pg LDFLAGS=-pg
```

The updated makefiles can be found at
  [my github repository](https://github.com/Lunderberg/sample_makefiles).
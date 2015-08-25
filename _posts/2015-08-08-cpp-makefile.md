---
layout: post
title: Magic C++ Makefile
tags: [c++, make]
---

Makefiles are often touched on, but rarely described in depth.
It is easy to start a makefile,
  but it becomes quite difficult to keep it up to date.
Dependencies get missed, source files get missed,
  and the entire thing is a pain to read.
Much better, overall, to have `make` determine everything automatically.

To start with,
  let's make a simple Makefile to compile a small project.

```
.
|-- Makefile
|-- mainA.cc
|-- mainB.cc
|-- include
|   |-- A.hh
|   `-- B.hh
`-- src
    |-- A.cc
    `-- B.cc
```

Both `mainA.cc` and `mainB.cc` contain an `int main()`,
  and should be compiled into separate executables.
Each of `A.cc` and `B.cc` should be compiled into object files,
  which then

Below is a Makefile that constructs these executables.
For each entry, the target is listed, followed by the dependencies.
Then, the commands necessary to make this target are given.


```make
CFLAGS = -Iinclude

all: bin/mainA bin/mainB

bin/mainA: build/mainA.o build/src/A.o build/src/B.o
	mkdir -p bin
	g++ build/mainA.o build/src/A.o build/src/B.o -o bin/mainA

bin/mainB: build/mainB.o build/src/A.o build/src/B.o
	mkdir -p bin
	g++ build/mainB.o build/src/A.o build/src/B.o -o bin/mainB

build/mainA.o: mainA.cc include/A.hh include/B.hh
	mkdir -p build
	g++ -c $(CFLAGS) mainA.cc -o build/mainA.o

build/mainB.o: mainB.cc include/A.hh include/B.hh
	mkdir -p build
	g++ -c $(CFLAGS) mainB.cc -o build/mainB.o

build/src/A.o: src/A.cc include/A.hh
	mkdir -p build/src
	g++ -c $(CFLAGS) src/A.cc -o build/src/A.o

build/src/B.o: src/B.cc include/B.hh
	mkdir -p build/src
	g++ -c $(CFLAGS) src/B.cc -o build/src/B.o

clean:
	rm -rf bin build
```

This works, but isn't very flexible.
We are manually listing all the dependencies.
The dependencies are used to determine which targets need to be rebuilt,
  rather than rebuilding everything, every time.
If we were to add an include file somewhere, or to remove one, we would need to edit the Makefile.
As time goes on, I would be likely to forget, at which point something might not be recompiled
  when it needs to be.

The solution is to use a dependency file.
`g++` has a flag `-MMD`, which will output all dependencies in a form that `make` can understand.
We then include those dependency files, allowing all dependencies to be known
  and to be automatically updated whenever the code is changed.

```make
CFLAGS += -MMD
-include $(shell find build -name "*.d" 2> /dev/null)
```

In addition, there is quite a bit of repetition.
We can reduce this repetition by using some of the automatic variables provided by `make`.
A full list of variables available can be found
  [here](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html).
Here are the variables that we will be using.

* `$@`, the current target
* `$(@D)`, the directory of the current target
* `$<`, the first prerequisite
* `$^`, a list of all prerequisites

With these in hand, the Makefile becomes much cleaner.

```make
CFLAGS = -Iinclude -MMD

all: bin/mainA bin/mainB

bin/mainA: build/mainA.o build/src/A.o build/src/B.o
	mkdir -p bin
	g++ $^ -o $@

bin/mainB: build/mainB.o build/src/A.o build/src/B.o
	mkdir -p bin
	g++ $^ -o $@

build/mainA.o: mainA.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

build/mainB.o: mainB.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

build/src/A.o: src/A.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

build/src/B.o: src/B.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

-include $(shell find build -name "*.d" 2> /dev/null)

clean:
	rm -rf bin build
```

Notice how, with the use of the automatic variables,
  many of our rules have identical structures.
We can take advantage of this to write pattern rules.
Whenever a target is needed, but is not explicitly listed,
  `make` will look through all pattern rules available to see if one matches.
If it does, then that pattern rule is used to generate the target.
Below, see how we can now build any of our `.o` files with just a single rule,
  instead of listing them all out.

```make
CFLAGS = -Iinclude -MMD

all: bin/mainA bin/mainB

bin/%: build/%.o build/src/A.o build/src/B.o
	mkdir -p bin
	g++ $^ -o $@

build/%.o: %.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

-include $(shell find build -name "*.d" 2> /dev/null)

clean:
	rm -rf bin build
```

Next, I want to get away from listing any of the files myself.
The Makefile should be smart enough to find my source files.
This way, if I add any, they get added to the Makefile automatically.
I use `wildcard` to find all the source files,
  and `patsubst` to determine which `.o` files should be built from them.

```make
CFLAGS = -Iinclude -MMD

EXE_SRC_FILES = $(wildcard *.cc)
EXECUTABLES = $(patsubst %.cc,bin/%,$(EXE_SRC_FILES))
SRC_FILES = $(wildcard src/*.cc)
O_FILES = $(patsubst %.cc,build/%.o,$(SRC_FILES))

all: $(EXECUTABLES)

bin/%: build/%.o $(O_FILES)
	mkdir -p bin
	g++ $^ -o $@

build/%.o: %.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

-include $(shell find build -name "*.d" 2> /dev/null)

clean:
	rm -rf bin build
```

We can now add additional `.cc` files, either in the main directory,
  or in the `src` directory, and the makefile will react accordingly.

A commented version of this makefile can be found at
  [my github repository](https://github.com/Lunderberg/sample_makefiles/blob/8e30d524d5c6922bcabfd0dd583e01225dcb3953/Makefile.simple).
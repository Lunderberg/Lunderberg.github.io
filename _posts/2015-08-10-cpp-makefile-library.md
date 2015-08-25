---
layout: post
title: C++ Makefile, Shared Libraries
tags: [c++, make]
---

In my [previous post]({% post_url 2015-08-08-cpp-makefile %}),
  I described a makefile that automatically determines
  which source files need to be built.
Here, I will expand on this makefile to include shared libraries.

Standalone executables are very convenient.
They can be moved anywhere, and run,
  without needing to worry about dependencies.
However, if multiple executables all use the same code,
  standalone executables use more disk space,
  whereas a shared library does not store duplicate copies of the code.
In addition, if multiple executables use the same shared library,
  it only needs to be loaded once, which saves RAM usage.

Starting from the previous makefile,
  we only need to make a few changes.
First, we need an additional flag when compiling our object files.

```make
CFLAGS += -fPIC
```

`PIC` stands for "Position-Independent Code".
Usually, function locations and jumps will be given absolute addresses.
With this flag, these are instead given as relative locations.
This is needed in a shared library,
  since the library doesn't know where it will be loaded into memory.

Next, we add a rule to compile the shared library itself.

```make
LIB_NAME = MyLibrary
lib/lib$(LIB_NAME).so: $(SRC_FILES)
	 g++ $^ -shared -Wl,-soname,lib$(LIB_NAME).so -o $@
```

This specifies a library to make,
  and the files from which it should be constructed.
We leave the library name itself as a variable,
  so that it can be easily adjusted later on.

Finally, we must add the library to the compilation of the executables.

```make
LFLAGS = -Llib -l$(LIB_NAME) -Wl,-rpath,\$$ORIGIN/../lib
bin/%: build/%.o | lib/lib$(LIB_NAME).so
	mkdir -p $(@D)
	g++ $< $(LFLAGS) -o $@
```

There are three extra flags that we use here.
The first, `-Llib`, tells the compiler
  that it can find shared libraries in the `lib` folder.
The second, `-l$(LIB_NAME)`, tells the compiler
  that it should look for unresolved symbols in `lib$(LIB_NAME).so`.
Finally, the last argument, `-Wl,-rpath,\$$ORIGIN/../lib`
  tells the runtime environment where to look for shared libraries.
The `\$$ORIGIN` is a special variable that will expand
  to the location of the executable at runtime.
This way, so long as you move both your `bin` and `lib` directories together,
  the resulting executables can be moved wherever you want.

In addition, we specify the library as an "order-only dependency".
The library must be built before building the executable.
However, if the library is re-built, the executable does not need to be rebuilt.

Now, putting this all together.

```make
LIB_NAME = MyLibrary
CFLAGS = -Iinclude -fPIC
LFLAGS = -Llib -l$(LIB_NAME) -Wl,-rpath,\$$ORIGIN/../lib

EXE_SRC_FILES = $(wildcard *.cc)
EXECUTABLES = $(patsubst %.cc,bin/%,$(EXE_SRC_FILES))
SRC_FILES = $(wildcard src/*.cc)
O_FILES = $(patsubst %.cc,build/%.o,$(SRC_FILES))

all: $(EXECUTABLES) lib/lib$(LIB_NAME).so

bin/%: build/%.o | lib/lib$(LIB_NAME).so
	mkdir -p $(@D)
	g++ $< $(LFLAGS) -o $@

lib/lib$(LIB_NAME).so: $(O_FILES)
	mkdir -p $(@D)
	g++ $^ -shared -Wl,-soname,lib$(LIB_NAME).so -o $@

build/%.o: %.cc
	mkdir -p $(@D)
	g++ -c $(CFLAGS) $< -o $@

clean:
	rm -rf bin build lib

CFLAGS += -MMD
-include $(shell find build -name "*.d" 2> /dev/null)
```

This will take all the source files present in the `src` directory,
  compile them into a shared library,
  then build all executables based on that library.
A commented version of this makefile can be found at
  [my github repository](https://github.com/Lunderberg/sample_makefiles/blob/8e30d524d5c6922bcabfd0dd583e01225dcb3953/Makefile.single_library).
---
layout: post
title: C++ Makefile, Multiple Libraries
tags: [c++, make]
---

Previously, we made a makefile to compile programs against a single library.
Now, we will extend that makefile to compile multiple libraries.

First, we want to make a few variables to determine the compilation flags,
  once we have the libraries made.

```make
LIBRARY_FOLDERS   = $(wildcard lib?*)
LIBRARY_OUTPUT    = $(patsubst %,lib/%.so,$(LIBRARY_FOLDERS))
LIBRARY_INCLUDES  = $(patsubst %,-I%/include,$(LIBRARY_FOLDERS))
LIBRARY_FLAGS     = $(patsubst lib%,-l%,$(LIBRARY_FOLDERS))
```

First, this finds all folders that start with `lib`.
Everything after this will be the name of the shared library.
After that, we create the output filenames,
  and the compiler flags for the header file locations
                             and the libraries
We add the flags to appropriate variables,
  to pass them to the compiler.
As before, in case the user has specified additional options,
  we add the `override` keyword.

```make
override CPPFLAGS += $(LIBRARY_INCLUDES)
override LDLIBS   += $(LIBRARY_FLAGS)
```

Now, we will create functions to determine which `.o` files
  should be compiled to each library.

```make
library_src_files = $(wildcard lib$(1)/src/*.cc)
library_o_files   = $(patsubst %.cc,build/%.o,$(call library_src_files,$(1)))
```

Note the use of the `$(call)` function, and the `$(1)` variable.
This is how function calls work in makefiles.
The `$(1)` is expanded to the first variable passed,
  `$(2)` to the second variable, and so on.
Now, given a library name, we can determine all the object files necessary.

```make
.SECONDEXPANSION:

lib/lib%.so: $$(call library_o_files,%)
	mkdir -p $(@D)
	$(CXX) $^ -shared -o $@
```

Note that there are two dollar signs in the call, not just one.
A single dollar sign will perform variable replacement immediately.
Two dollar signs will delay the variable replacement until the target of the rule is known.
This allows us to call the `library_o_files` function once for each library.
The `.SECONDEXPANSION:` line is needed to enable this feature.

Now, each folder starting with `lib` will be compiled into a separate shared library.
Each include folder is added to the compiler flags.
The full makefile, complete with comments, is available on
  [github](https://github.com/Lunderberg/sample_makefiles/blob/master/Makefile.multiple_libraries).
---
layout: post
title: C++ Makefile, Static Library
tags: [c++, make]
date: 2015-08-26 19:00
---

All of our previous examples have used only shared libraries.
While shared libraries are very useful, static libraries also have their use.
Link-time optimization can be done much more aggressively.
Any functions that are not called can be pruned by the compiler,
  whereas they cannot be pruned from a shared library.
Here, we will improve the makefile to compile either shared or static libraries.

First, let's add variables to indicate whether we should build a static or a shared library.

```make
BUILD_SHARED = 1
BUILD_STATIC = 0
```

If `BUILD_SHARED` is non-zero, we will build a shared library.
If `BUILD_STATIC` is non-zero, we will build a static library.
If both are non-zero, we will make both libraries.
We will link the executables against whichever library's value is greater.

Now, we define each library that we want to make.

```make
ifneq ($(BUILD_SHARED),0)
    SHARED_LIBRARY_OUTPUT = $(patsubst %,lib/%.so,$(LIBRARY_FOLDERS))
endif

ifneq ($(BUILD_STATIC),0)
    STATIC_LIBRARY_OUTPUT = $(patsubst %,lib/%.a,$(LIBRARY_FOLDERS))
endif
```

Next, we want to define a different rule for building the object files.
When compiling a shared library, we want the `-fPIC` flag,
  whereas we don't want it when compiling a static library.
The object files to go into a static library will end in `.o`,
  and object files to go into a shared library will end in `.os`.

```make
build/$(BUILD)/build/%.os: %.cc
	mkdir -p $(@D)
	$(CXX) -c -fPIC $(CPPFLAGS) $(CXXFLAGS) $< -o $@

build/$(BUILD)/build/%.o: %.cc
	mkdir -p $(@D)
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

Now, we add a rule for each of the libraries.
We will add a variable `AR` to allow the user to select which archiver to use.
In addition, we define `library_os_files` in terms of `library_o_files`.

```make
AR = ar
library_os_files  = $(addsuffix s,$(call library_o_files,$(1)))

build/$(BUILD)/lib/lib%.a: $$(call library_o_files,%)
	mkdir -p $(@D)
	$(AR) rcs $@ $^

build/$(BUILD)/lib/lib%.so: $$(call library_os_files,%)
	mkdir -p $(@D)
	$(CXX) $(LDFLAGS) $^ -shared $(SHARED_LDLIBS) -o $@
```

Finally, we want to have a rule to compile the executables.
We will make two versions of the rule,
  depending on whether we are linking against the shared or the static libraries.

```make
ifeq ($(shell test $(BUILD_SHARED) -gt $(BUILD_STATIC); echo $$?),0)
build/$(BUILD)/bin/%: build/$(BUILD)/build/%.o $(O_FILES) | $(SHARED_LIBRARY_OUTPUT)
	mkdir -p $(@D)
	$(CXX) $(ALL_LDFLAGS) $^ $(ALL_LDLIBS) -o $@
else
build/$(BUILD)/bin/%: build/$(BUILD)/build/%.o $(O_FILES) $(STATIC_LIBRARY_OUTPUT)
	mkdir -p $(@D)
	$(CXX) $(ALL_LDFLAGS) $^ $(ALL_LDLIBS) -o $@
endif
```

Now, we can compile the code to either a shared or a static library,
  depending on what is needed for a particular project.
The full makefile can be found on
  [github](https://github.com/Lunderberg/sample_makefiles/blob/a608e5979d650247dfcf994fc3863982f5e1fe5b/Makefile).

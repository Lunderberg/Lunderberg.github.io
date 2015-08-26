---
layout: post
title: C++ Makefile, Build Targets
tags: [c++, make]
date: 2015-08-25 20:00
---

Currently, we can change build parameters from the command line by overwriting the makefile's variables.
However, if we are making a large number of changes, this becomes unwieldy.
We wouldn't want to need to remember all the parameters for switching
  between a debug build and a release build, for example.
Let's add the ability to switch between different sets of parameters.

First, let's define a new parameter, `BUILD`.
This will serve as a unique identifier for each set of parameters.

```make
BUILD = default
```

Remember that this variable can be overridden on the command line.
Therefore, we can check to see whether it has been overridden by comparing it to the value we just gave.
If it has been, we will include a file with the custom variables.

```make
ifneq ($(BUILD),default)
    include build-targets/$(BUILD).inc
endif
```

Now, whatever variables we define in the `.inc` file will be propagated through.
For example, we could make a `profile.inc` target that will profile the code with `gprof`.

```make
CPPFLAGS += -pg
LDFLAGS  += -pg
```

We could then compile with `make clean && make BUILD=profile` to recompile the entire project.
The `profile.inc` file is included, and the extra flags are added.

This works, but requires a complete recompiling with each change.
If I am switching repeatedly between two particular builds,
  I don't want to wait for a full recompile each time.
Therefore, let's store the intermediate files in different folders for each build type.
The changes for a few of the rules are shown below,
  with all the rest behaving similarly.

```make
SRC_FILES = $(wildcard src/*.cc)
O_FILES = $(patsubst %.cc,build/$(BUILD)/build/%.o,$(SRC_FILES))

build/$(BUILD)/bin/%: build/$(BUILD)/build/%.o $(O_FILES) | $(LIBRARY_OUTPUT)
	mkdir -p $(@D)
	$(CXX) $(LDFLAGS) $^ $(LDLIBS) -o $@

build/$(BUILD)/build/%.o: %.cc
	mkdir -p $(@D)
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $< -o $@
```

Now we have our separate builds, with each build generating files in
  `build/$(BUILD)/bin` and `build/$(BUILD)/lib`.
We want to copy these files to `bin` and `lib`.
The naïve solution would be to add a rule to copy these.

```make

bin/%: build/$(BUILD)/bin/%
	mkdir -p $(@D)
	cp -f $< $@

lib/%: build/$(BUILD)/lib/%
	mkdir -p $(@D)
	cp -f $< $@
```

However, this would not behave correctly when the build target changes.
Make will update a target if the timestamp of a prerequisite is newer than that of the target.
Therefore, we might not copy all of the executables and libraries of the new build into the main folder.

To get around this, we make a separate target which will contain the current build target.
By giving it a prerequisite that is `PHONY`, we will always try to rebuild this file.
However, we only update the file if the `BUILD` variable is different from its previous value.
Then, we make our copy rule depend on this file.
If the `BUILD` variable changes, then so does the dummy file, and so each output file is copied in.

```make
.PHONY: force

.build-target: force
	echo $(BUILD) | cmp -s - $@ || echo $(BUILD) > $@

bin/%: build/$(BUILD)/bin/% .build-target
	mkdir -p $(@D)
	cp -f $< $@

lib/%: build/$(BUILD)/lib/% .build-target
	mkdir -p $(@D)
	cp -f $< $@
```

The full makefile can be found on
  [github](https://github.com/Lunderberg/sample_makefiles/blob/be5fbf0a5f3f633c7b4a8681255c2cd2b1f563cd/Makefile)

---
layout: post
title: C++ Makefile, Library-Specific Flags
tags: [c++, make]
date: 2015-08-24 21:30
---

In the [previous post]({% post_url 2015-08-24-cpp-makefile-multiple-libraries %}),
  we made a makefile capable of compiling multiple shared libraries,
  and multiple executables.
Now, modifying the makefile so that each library can have specific compilation flags.

Makefiles allow for variables to be target-specific variables to be specified.
For example, if a specific library needs to be compiled under C++03,
  while the rest of the project requires C++11,
  we could overwrite the `CXXFLAGS` variable for those specific targets.

```make
build/libLibrary03/%: override CXXFLAGS += -std=c++03
```

This shouldn't be in the makefile, though, because I want it to remain as general as possible.
Instead, let's define a few variables inside an include file.

```make
CPPFLAGS_EXTRA =
CXXFLAGS_EXTRA = -std=c++03
LDFLAGS_EXTRA  =
SHARED_LDLIBS  =
```

Now, we include that in our makefile and set the target-specific variables.

```make
-include libLibrary03/Makefile.inc
build/libLibrary03/%.o: override CPPFLAGS := $(CPPFLAGS) $(CPPFLAGS_EXTRA)
build/libLibrary03/%.o: override CXXFLAGS := $(CXXFLAGS) $(CXXFLAGS_EXTRA)
lib/libLibrary03.so:  override LDFLAGS  := $(LDFLAGS)  $(LDFLAGS_EXTRA)
lib/libLibrary03.so:  override SHARED_LDLIBS := $(SHARED_LDLIBS)
```

Note how we can define target-specific variables for each a single target file,
  or for a pattern rule.
This lets us easily add the `CXXFLAGS` to any `.o` file present in the library.
However, this is still specific to one library.
To make it more general, let's define a template that will be used for any library.

```make
define library_variables
CPPFLAGS_EXTRA =
CXXFLAGS_EXTRA =
LDFLAGS_EXTRA  =
SHARED_LDLIBS  =
-include $(1)/Makefile.inc
build/$(1)/%.o: override CPPFLAGS := $$(CPPFLAGS) $$(CPPFLAGS_EXTRA)
build/$(1)/%.o: override CXXFLAGS := $$(CXXFLAGS) $$(CXXFLAGS_EXTRA)
lib/$(1).so:  override LDFLAGS  := $$(LDFLAGS)  $$(LDFLAGS_EXTRA)
lib/$(1).so:  override SHARED_LDLIBS := $$(SHARED_LDLIBS)
endef
```

Now, we evaluate the template for each library that we are going to make.

```make
$(foreach lib,$(LIBRARY_FOLDERS),$(eval $(call library_variables,$(lib))))
```

With this, we can define additional flags for each library, without needing to edit the makefile itself.
As before, the full makefile can be found on
  [github](https://github.com/Lunderberg/sample_makefiles/blob/master/Makefile.multiple_libraries).
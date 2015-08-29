---
layout: post
title: C++ Makefile, Library Folders
tags: [c++, make]
date: 2015-08-29 08:00
---

[Earlier]({% post_url 2015-08-24-cpp-makefile-library-specific-flags %}),
  we allowed each library to have individual compilation flags.
Here, we will re-implement this feature in a more flexible way,
  and allow libraries to specify multiple directories that contain source code.

First, we modify `library_src_files` gets modified to accept a second parameter,
  containing the directories that should be searched for source files.

```make
library_src_files = $(foreach src_dir,$(2),$(call find_in_dir,$(1)/$(src_dir),$(CPP_EXT) $(C_EXT)))
library_o_files   = $(call o_file_name,$(call library_src_files,$(1),$(2)))
library_os_files   = $(addsuffix s,$(call library_o_files,$(1),$(2)))
```

The next part is a large chunk, but we'll break it down into smaller pieces afterward.
This defines a function, calls it, then evaluates the result multiple times.

```make
define library_commands

    ifneq ($$(BUILD_STATIC),0)
       STATIC_LIBRARY := $$(call STATIC_LIBRARY_NAME,$(1))
    else
       STATIC_LIBRARY :=
    endif

    ifneq ($$(BUILD_SHARED),0)
       SHARED_LIBRARY := $$(call SHARED_LIBRARY_NAME,$(1))
    else
       SHARED_LIBRARY :=
    endif

    LIBRARY = $$(SHARED_LIBRARY) $$(STATIC_LIBRARY)
    LIBRARY_SRC_DIRS = src

    -include $(1)/Makefile.inc

    build/$$(BUILD)/$$(call SHARED_LIBRARY_NAME,$(1)): $$(call library_os_files,$(1),$$(LIBRARY_SRC_DIRS))
	mkdir -p $(@D)
	$$(CXX) $$(ALL_LDFLAGS) $$^ -shared $$(SHARED_LDLIBS) -o $$@

    build/$$(BUILD)/$$(call STATIC_LIBRARY_NAME,$(1)): $$(call library_o_files,$(1),$$(LIBRARY_SRC_DIRS))
	mkdir -p $(@D)
	$$(AR) rcs $$@ $$^
endef

$(foreach lib,$(LIBRARY_FOLDERS),$(eval $(call library_commands,$(lib))))
```

First, we define three variables, `STATIC_LIBRARY`, `SHARED_LIBRARY`, and `LIBRARY`.
These point to the library that is currently being made.
Then, library-specific variables can be defined as target-specific variables on these targets.

```make
$(LIBRARY): ALL_CPPFLAGS += -DMY_DEFINE
```

We also define a default value for `LIBRARY_SRC_DIRS`, which can be overridden by the include file.

Finally, we define targets for the library to be generated.
Here, we expand the calls to `library_o_files` to determine which files need to be included.
This uses `LIBRARY_SRC_DIRS`, searching for src files only in the directories specified.

The full makefile can be found on
  [github](https://github.com/Lunderberg/sample_makefiles/tree/bcdef6e57516bf06133cced8b382fb4dea0d79a2).
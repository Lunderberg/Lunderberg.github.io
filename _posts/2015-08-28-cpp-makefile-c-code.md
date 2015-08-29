---
layout: post
title: C++ Makefile, C code
tags: [c++, make]
---

So far, we have dealt with C++ code exclusively.
Today, I was looking at the makefile, and I realized that it would be quite simple to compile C code as well.
In addition, we will generalize the code, so that our C++ files can have file extensions other than `.cc`.

First, we need to define the rules to compile C code into object files.
These are nearly identical to the rules to compile C++ code.
The only difference is that we use the `$(CC)` variable for the compiler,
  and we pass it flags from `$(CFLAGS)` instead of `$(CXXFLAGS)`.

```make
CC = gcc

build/$(BUILD)/build/%.o: %.c
	@$(call run_and_test,$(CC) -c $(ALL_CPPFLAGS) $(ALL_CFLAGS) $< -o $@,Compiling)

build/$(BUILD)/build/%.os: %.c
	@$(call run_and_test,$(CC) -c $(PIC_FLAG) $(ALL_CPPFLAGS) $(ALL_CFLAGS) $< -o $@,Compiling)
```

Now, all that is needed is to indicate that those `.o` files are generated,
  and are included in the appropriate libraries.

```make
find_in_dir = $(foreach ext,$(2),$(wildcard $(1)/*.$(ext)))
o_file_name = $(foreach file,$(1),build/$(BUILD)/build/$(basename $(file)).o)

EXE_SRC_FILES = $(wildcard *.$(CPP_EXT)) $(wildcard *.$(C_EXT))
SRC_FILES = $(call find_in_dir,src/,$(CPP_EXT) $(C_EXT))
O_FILES = $(call o_file_name,$(SRC_FILES))

library_src_files = $(call find_in_dir,lib$(1)/src/,$(CPP_EXT) $(C_EXT))
library_o_files   = $(call o_file_name,$(call library_src_files,$(1)))
```

And done.

That was rather short, so let's extend it a bit more.
Instead of hard coding the file extension, let's allow it to be user-defined.
Also, let's allow there to be more than one.
We will define a variable, then use `$(eval ...)` to transform it into a rule.

```make
C_EXT = c
CPP_EXT = C cc cpp cxx c++ cp

define C_BUILD_RULES
build/$$(BUILD)/build/%.o: %.$(1)
	@$$(call run_and_test,$$(CC) -c $$(ALL_CPPFLAGS) $$(ALL_CFLAGS) $$< -o $$@,Compiling)

build/$$(BUILD)/build/%.os: %.$(1)
	@$$(call run_and_test,$$(CC) -c $$(PIC_FLAG) $$(ALL_CPPFLAGS) $$(ALL_CFLAGS) $$< -o $$@,Compiling)
endef

$(foreach ext,$(C_EXT),$(eval $(call C_BUILD_RULES,$(ext))))
```

Note that the variables inside the definition have two dollar signs preceding them.
This is because they are evaluated twice.
Once in the call to `$(eval ...)`, and once when the rule is used.

This isn't needed much for C files, since most will use the `.c` extension.
For C++, there are quite a few extensions, and so it is useful to be able to compile them all.

The makefile can be found on
  [github](https://github.com/Lunderberg/sample_makefiles/tree/196d9ce6a3a54174ca420105b5594fb28adde852).
---
layout: post
title: C++ Makefile, Cross-Compiling
tags: [c++, make]
---

I have written one [previous post]({% post_url 2014-12-30-cross-platform-cpp %}) on cross compiling,
  which used the scons build system.
However, scons is not universally installed, and has some issues with larger projects.
Here, I will use the same strategy for cross-compiling using mingw-w64,
  but with the generic makefile that has been developed over the last several posts.

We will incorporate cross-compiling as a separate build target.
That way, to make the standard binary, we will use `make`,
  and to cross-compile, we will use `make BUILD=win32` and `make BUILD=win64`
  for 32-bit and 64-bit binaries.
The main meat of this has already been accomplished when we make the
  [separate build targets]({% post_url 2015-08-25-cpp-makefile-build-targets %}),
  and so we will just need to add a bit more flexibility.

Mingw-w64 contains versions of gcc that target windows systems,
  along with all the dynamic libraries needed to run on windows.
So long as we switch compilers to the appropriate mingw-w64 provided executable,
  we will produce windows binaries instead of native binaries.

```make
# In win32.inc
CXX = i686-w64-mingw32-g++
AR = i686-w64-mingw32-ar
```

We want to follow standard conventions for filenames on windows.
Executables should end in `.exe`,
  shared libraries should be named `MyLibrary.dll` instead of `libMyLibrary.so`.
In addition, because windows does not have the equivalent of `RPATH`,
  shared libraries are usually installed either system-wide
  or placed into the same folder as the executable.
Since it is ridiculous to assume that everybody wants to install a program system-wide,
  we will be placing the shared libraries in the same folder as the executable.
We define a few variables to describe the output file names.

```make
# In win32.inc
EXE_NAME = bin/$(1).exe
SHARED_LIBRARY_NAME = $(patsubst lib%,bin/%.dll,  $(1))
STATIC_LIBRARY_NAME = $(patsubst lib%,lib/%.dll.a,$(1))
```

Now, we modify our targets and our build rules to make calls to these functions.
Here are shown the modifications for the shared libraries.
The rules for the executables and the static libraries are done in a similar manner.

```make
# In Makefile
SHARED_LIBRARY_OUTPUT = $(foreach lib,$(LIBRARY_FOLDERS),$(call SHARED_LIBRARY_NAME,$(lib)))

$(call SHARED_LIBRARY_NAME,lib%): build/$(BUILD)/$(call SHARED_LIBRARY_NAME,lib%) .build-target
	mkdir -p $(@D)
	cp -f $< $@

build/$(BUILD)/$(call SHARED_LIBRARY_NAME,lib%): $$(call library_os_files,%)
	mkdir -p $(@D)
	$(CXX) $(ALL_LDFLAGS) $^ -shared $(SHARED_LDLIBS) -o $@
```

Finally, getting rid of a minor annoyance.
Whenever gcc (or mingw-w64, which is based on gcc) is producing an object file for windows,
  it issues a warning if the `-fPIC` flag is present.
This is because all code on this target is position-independent, and therefore `-fPIC` does nothing.
Rather than just ignoring the flag, a warning is produced.
Therefore, we move the flag into a variable, so that our build target can disable its use.

```make
# In win32.inc
PIC_FLAG =

# In Makefile
build/$(BUILD)/build/%.os: %.cc
	mkdir -p $(@D)
	$(CXX) -c $(PIC_FLAG) $(ALL_CPPFLAGS) $(ALL_CXXFLAGS) $< -o $@
```

The full version of this makefile can be found on
  [github](https://github.com/Lunderberg/sample_makefiles/tree/1faf6f33f52706855f71012dad5c12c44e9fc933).
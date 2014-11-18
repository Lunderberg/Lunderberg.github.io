---
layout: post
title: Minimal ROOT Example
tags: [c++, root]
---

ROOT is a C++ data analysis framework designed by CERN.
It's greatest strength is in writing arbitrary C++ objects to file
without needing to write serialization code for those objects.

However, it takes a bit of fiddling to get it to work.
Many magic incantations are needed, and error messages tend to be unenlightening.
To avoid re-inventing the wheel,
I made a minimal example capable of writing an arbitrary data structure to a ROOT tree.

If you would like to just jump to the chase,
clone the repository at http://github.com/Lunderberg/minimal_root ,
and start editing.
What follows is an explanation of what is needed.

First, the definition of the class to be written to the file.

```c++
#include "TObject.h"

class DataStructure : public TObject{
public:
  DataStructure();
  ClassDef(DataStructure,1);

private:
  double x,y;
};
```

The data structure to be written must be a subclass of the ROOT class `TObject`.
In addition, it must have a public constructor that takes no parameters.
The `ClassDef` statement is a macro provided by ROOT that must be included in each class to be written.
In addition, it must be specified in a `public` section of the class.
Each instance variable will be written to the data file,
even though they are `private` variables.

Next, there is a `LinkDef.h` file.

```c++
#ifdef __CINT__
#pragma link off all globals;
#pragma link off all classes;
#pragma link off all functions;
#pragma link C++ class DataStructure;
#endif
```

In order for ROOT to write the data members,
  it first needs to analyze the class structure to determine how the class should be written.
By running `rootcint -f Dictionary.cc DataStructure.hh LinkDef.h`,
  `Dictionary.cc` and `Dictionary.h` files are created.
These files contain a full listing of the class structure,
  with all members variables are listed by name.
This allows them to later be listed by name.

Finally, the `Program.cc` file defines a data structure,
  creates a tree with that data structure as a branch,
  then fills it.

The Makefile included is intended to remove as much of the headache as possible.
Each `cc`-file in the base directory as a separate executable,
  placed into the bin directory.
Each `cc`-file inside the src directory will be compiled into a shared library,
  also placed into the bin directory.
This shared library can be loaded into the root interpreter,
  allowing for interaction with the custom classes.
All dependencies between files are tracked,
  so that any modification will result in only the dependent files being recompiled.
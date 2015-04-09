---
layout: post
title: My Preferred C++ Style
tags: [c++]
---

A friend recently asked me for advice in how to structure `C++` code.
It made me need to think through what rules I tend to follow.

Here, I separate things out into three main different priorities.

* Priority A guidelines are useful to follow in all circumstances.
  When writing new code or modifying existing code,
    the code should follow these guidelines once I am done.

* Priority B guidelines are useful to follow,
    but only if followed throughout the entire codebase.
  When starting a new project, or when refactoring code,
    I follow these guidelines.
  However, if a codebase does not currently follow these guidelines,
    it is better to follow whatever convention is currently followed.

* Priority C guidelines are trivialities.
  Pick your favorite style, and use it consistently.
  If there is an existing convention, use it instead.
  These are matters of personal style,
    but it is best to have consistency throughout the codebase.

File Organization
-----------------

* Priority B.
  Every source file that does not include `int main()` should be in the `src` directory.
  Every header file should be in the `include` directory.

* Priority A.
  All source files should be named identically to the corresponding header file,
    with the exception of the file extension.

* Priority C.
  Pick a file extension for the header files, typically `.hh` or `.hpp`.
  Pick a file extension for the source files, typically `.cc` or `.cpp`.
  Use them consistently.

* Priority A.
  Every class gets its own source and header file.
  I will sometimes make an exception for very, very tightly coupled classes
  and put them in the same source and header file.

* Priority B.
  A group of related files may share a source and header file.

Function/Method Organization
----------------------------

* Priority A.
  Every function/method should be able to have a short name.
  If a function cannot be described with a short name,
  it should be split into smaller functions that can.

* Priority A.
  Every function/method should be able to be displayed in its entirety on a monitor,
    using a normal font size.
  It it cannot, then the function should be split into subordinate functions.
  If these subordinate functions should be hidden from users of the library,
    then it should be defined in the source file with no corresponding declaration in the header file.

* Priority A.
  Every function should depend only on its parameters.
  There should be no global state, and it should be safe to call many times.
  If the same parameters are passed in, then it should return the same value.

* Priority A.
  Every method should depend only on its parameters and the state of the class.

* Priority A.
  If a function requires additional state,
    it should be changed into a method within a class.
  The class then holds the additional state.

* Priority B.
  `const` should be used whenever possible in function arguments.
  This allows one to consider the effect of a function more easily.
  I would put this at Priority A, except that sometimes,
    due to poor code organization,
    it is impossible to add const without breaking the build.

* Priority A.
  Functions should have as few arguments as possible.
  If many functions have similar parameters,
    the parameters should be grouped together as a struct,
    which contains the shared parameters.

* Priority B.
  Pass large objects by const reference, to avoid unnecessary copying.

* Priority B.
  If a function or method needs to return multiple values,
    it should be done through returning a struct or a `std::tuple`.

Class Organization
------------------

* Priority A.
  Any methods of a class can be called in any order.
  There should never be any undefined behavior invoked
    that results from methods being called in an unexpected order.
  If setup code is required, it should be performed in the constructor.
  Any errors in setup code should be reported as exceptions.

* Priority A.
  Resources should be held by a class, which releases the resource in its destructor.
  That is, follow the RAII pattern,
    which ensures that all resources are released,
    regardless of whether the function returns or throws an exceptions.
  In addition, it prevents the programmer from forgetting to release a resource.
  Resources include memory allocation, creation of temporary files,
    network connections, etc.
  If using `C++11`, pointers should always be held
    as `std::unique_ptr` or `std::shared_ptr`.

* Priority B.
  structs should have only public member variables,
    and classes should have only private member variables.
  The purpose of structs is to be held or passed around,
    while the purpose of classes is to act directly.
  In C++, there is no fundamental difference between classes and structs.
  I find this to be a useful semantic difference.

* Priority A.
  Operator overloading of `+`, `-`, `*`, and `/` should be used
    when the object can be interpreted as a numeric type.
  For example, when adding two matrices together.
  It should not be used in other cases, such as adding a user to a database.

Error Handling
--------------

* Priority B.
  Use exceptions to handle errors.
  This one has some debate on it.
  For me, the main difference is the default behavior if accidentally ignored.
  An accidentally ignored exception will cause the program to halt at the point of failure,
    allowing it to be quickly noticed and fixed.
  An accidentally ignored error code will cause bugs, some obvious, some subtle,
    at a point other than the point of failure.

* Priority A.
  Only things inheriting from `std::exception` should be thrown.
  The exception should be thrown by value and caught by reference.

* Priority A.
  All exceptions thrown by a library should inherit from a single base class.
  This allows users of the library to catch any exception thrown by the library.

Source/Header File Organization
-------------------------------

* Priority A.
  Use forward declarations whenever possible, rather than `#include`.
  Having an include in the source file, rather than the header file,
    will speed up compilation,
    because a change in a header file will result in fewer files needing to be re-compiled.

* Priority A.
  Only use the `#include` statements that are needed.
  Order the includes to have the standard library first,
    then any external libraries, then your own include files.
  The one exception is that a source file
    should include its corresponding header file first.
  This ensures that any missing include files are found immediately.

* Priority A.
  Include files should never be order-dependent.

Project Organization
--------------------

* Priority A.
  Everything should be under source control.
  With distributed version control systems,
    it is quick and easy to place a directory under source control,
    even just for your personal benefit.

* Priority B.
  If code is re-used between multiple projects,
    it should be separated out as an independent library.
  In this case, every function and every class exposed should be part of a single namespace.

* Priority A.
  Commit early, commit often.
  Commit the code whenever you have something new that works.
  Never commit code that does not compile,
    and never commit code that does not pass unit tests.
  Fix these issues prior to commiting code.

* Priority A.
  Compile all code with `-Wall -Wextra -pedantic`.
  The code should compile with no warnings.

* Priority A.
  Use a reasonable makefile, or alternative build system.
  A change in any file should result in rebuilding all dependent files.
  `make clean` should never be required.
  You should be able to add and remove source and header files,
    add `#include` statements, and remove `#include` statements,
    all without requiring an edit to the makefile
    and without requiring anything other than calling `make`.
  This can be accomplished with wildcard rules and the `-MMD` flag.
  The `-MMD` flag should be used in preference to `makedepend`.

* Priority A.
  If generating shared libraries, do not require your users to set the `LD_LIBRARY_PATH`.
  Instead, us the `RPATH` linker options to point to the shared library.
  The `$ORIGIN` flag may be necessary as well.
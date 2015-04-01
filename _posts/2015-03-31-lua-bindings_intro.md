---
layout: post
title: Lua bindings, Introduction
tags: [c++, lua]
date: 2015-03-31 20:00
---

Embedded languages allow for easy extension of existing programs.
I like C++, but everything must be done at compile time.
Extension code, written in an embedded language and called from the C++ program,
  can be added or modified without needing to re-compile the main executable.
The embedded language may provide more flexibility, which makes development easier.
The extension code may be written by the user, to provide features that they want in the code.
And it has been something that I have been meaning to try out for a while,
  and there's no better time than now.

In choosing an embedded language, I had two main goals.
First, that it be easy to embed into any new program.
It should be easy to add to a new project,
  and should not require changing existing project code.
Second, especially for some of the plans that I have,
  I want to be able to safely run untrusted code.

I had considered Python as an embedded language.
`boost::python` is a magnificent library,
  which makes it fantastically easy to call Python code from C++,
  or C++ code from Python.
However, Python is notoriously difficult to sandbox.
Traditionally, when preparing an environment for untrusted code,
  one only provides functions that are safe, no matter what.
Calculating the sine of an angle is fine, no matter what the angle.
Deleting a file, on the other hand, is not.
Malicious code can, with some difficulty, gain access to the `__import__` function,
  and import the functions to delete files or open a web connection, for example.
As there is no consensus on a safe way to sandbox Python,
  I am avoiding it as an embedded language.

Lua, on the other hand, is designed to be an embedded language.
For a script to have access to a function,
  it must be explicitly given access.
Variable types exist that, for example, cannot be modified from within Lua,
  only from the surrounding C code.
For this reason,

The downside, I was unable to find existing bindings for Lua that would do exactly what I want.
In the spirit of discovery, I will be writing my own set of bindings.
These bindings are present on my [github page](http://github.com/Lunderberg/lua-bindings),
  and will be discussed here.

The series starts with [global variables]({% post_url 2015-03-31-lua-bindings_part-1_global-variables %}).
---
layout: post
title: C++, decltype and getter/setter
tags: [c++]
---

`decltype` is a new feature available in C++11.
It is generally used in templates,
  where the return type of a function may need to be determined from the input types.
When I heard of it, I thought that it might make getters and setters be easier to define.
After all, it would be nice to only need to state the type of a variable once.

```c++
class SomeClass{
public:
  decltype(var) GetVar(){ return var; }
  void SetVar(decltype(var) var_in){ var = var_in; }

private:
  int var;
};
```

The return type and parameter type of the getter and setter would then take the type
  of the internal variable.

Except that this doesn't work.
The compiler errors, stating that `var` is undefined at the line defining `GetVar()`.
I was confused, because I thought that everything inside a class was defined simultaneously.
After all, the following code compiles, even though `x` has not yet been defined.

```c++
class SomeOtherClass{
public:
  int GetX(){ return x; }
  int x;
};
```

As it turns out, my previous mental model was wrong.
Classes do not have all members defined simultaneously.
Rather, they are defined in two passes.
In the first pass, any method declarations and member variables are parsed.
In the second pass, all methods definitions are parsed.

`SomeOtherClass` can compile, because the first pass has defined `int x`,
  and it isn't used until the second pass.
However, `SomeClass` cannot compile, because `decltype(var)` cannot be determined prior to declaration of `var`.
This would work, if the variable declaration is done first, then the function declaration.

```c++
class SomeClass{
private:
  int var;

public:
  decltype(var) GetVar(){ return var; }
  void SetVar(decltype(var) var_in){ var = var_in; }
};
```

As neat as this is, I will keep it in the "interesting, and to be avoided" category.
There is something to be said for readable header files, even if it does reduce repetition.

As always, [StackOverflow](http://stackoverflow.com/a/16766850/2689797) provides marvelous explanations.

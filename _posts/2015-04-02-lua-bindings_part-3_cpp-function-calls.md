---
layout: post
title: Lua bindings, Calling C++ Functions
tags: [c++, lua]
---

As of [last time]({% post_url 2015-04-01-lua-bindings_part-2_lua-function-calls %}),
  we can set global variables, read global variables, and call Lua functions.
However, once inside Lua, we cannot make calls that return to C++.
These could be useful in many situations.
For example, numerical code can be written in C++ for greater speed,
  then called from Lua.
Alternatively, the Lua code may be a callback in a GUI framework,
  which needs to call C++ functions to change the state of the GUI.

Any C++ function called from within Lua must have a particular signature,
  and must assume a particular calling convention.
The arguments are pushed onto the Lua stack,
  then the function is called.
The function then acts, pushing its return values onto the stack.
Finally, the function returns the number of arguments to be returned to Lua.


As an example, to wrap the `average()` function defined below,
  I would need the following wrapper code.

```c++
double average(double a, double b){
  return (a + b)/2.0;
}

int lua_average(lua_State* L){
  double arg1 = Read<double>(L, 1);
  double arg2 = Read<double>(L, 2);
  double result = average(a, b);
  Push(L, result);
  return 1;
}
```

It could then be exposed to Lua as follows.

```c++
LuaState L;
L.SetGlobal("average", lua_average);
```

This works, but requires significant new code.
I want to avoid needing to rewrite the wrapper code for each new function.
To do this, rather than passing a function `lua_average`, I will pass in userdata.

Lua userdata is memory that is managed by Lua, but can only be modified in the surrounding program.
This could be used, for example, to store a pointer to a class.
That class could hold a function to be called.
First, let's set up the framework that we will need.

```c++
class LuaCallable{
public:
  virtual ~LuaCallable() { }
  virtual int call(lua_State* L) = 0;
};

int call_lua_callable(lua_State* L){
  void* storage = lua_touserdata(L, 1);
  LuaCallable* callable = *static_cast<LuaCallable**>(storage);
  lua_remove(L, 1);
  return callable->call(L);
}
```

Now, so long as the first argument to `call_lua_callable()` is a pointer to a LuaCallable,
  the function can be called.
This isn't good enough.
I want to be able to call functions directly.
My Lua code should not constantly be littered with calls to `call_lua_callable()`.

Instead, I will attach a metatable to the userdata.
Among other things, a metatable can specify a `__call` method,
  which allows the userdata to act as a function.
Let's make a `Push()` variant to attach a metatable.

```c++
void Push(lua_State* L, LuaCallable* callable){
  void* userdata = lua_newuserdata(L, sizeof(callable));
  *static_cast<LuaCallable**>(userdata) = callable;

  int metatable_uninitialized = luaL_newmetatable(L, "lua-bindings_cpp-function");
  if(metatable_uninitialized){
    Push(L, call_lua_callable);
    lua_setfield(L, -2, "__call");
  }
  lua_setmetatable(L, -2);
}
```

Now, the `LuaCallable` can be called directly.
All that is left is to make an implementation of the base class.

```c++
class LuaCallable_SimpleFunction : public LuaCallable {
public:
  LuaCallable_SimpleFunction(double (*func)(double,double)) : func(func) { }

  virtual int call(lua_State* L){
    double arg1 = Read<double>(L, 1);
    double arg2 = Read<double>(L, 2);
    double output = func(arg1, arg2);
    Push(L, output);
    return 1;
  }

private:
  double (*func)(double,double);
};
```

With this, the following code can provide the `average()` function to be used in Lua.

```c++
LuaState L;
L.SetGlobal("average", new LuaCallable_SimpleFunction(average));
```

This worked quite well.
Rather than needing to write a wrapper for each function,
  now a wrapper is only necessary for each function signature.
Let's use some templates to make even this step unnecessary.

```c++
template<typename RetVal, typename... Params>
class LuaCallable_CppFunction : public LuaCallable {
public:
  LuaCallable_CppFunction(RetVal (*func)(Params...)) : func(func) { }

  virtual int call(lua_State* L){
    return call_helper_function(build_indices<sizeof...(Params)>(), func, L);
  }

private:
  RetVal (*func)(Params...);

  template<int... Indices, typename RetVal_func>
  static int call_helper_function(indices<Indices...>, RetVal_func (*func)(Params...), lua_State* L){
    RetVal_func output = func(Read<Params>(L, Indices+1)...);
    Push(L, output);
    return 1;
  }

  template<int... Indices>
  static int call_helper_function(indices<Indices...>, void (*func)(Params...), lua_State* L){
    func(Read<Params>(L, Indices+1)...);
    return 0;
  }
};
```

Note the template magic with `indices` and `build_indices`.
These are necessary to extract call `Read()` with multiple different indices,
  and will be explained in a later post.

With this, it is now possible to write a generic `Push()` function that accepts any C++ function.

```c++
template<typename RetVal, typename... Params>
void Push(lua_State* L, RetVal (*func)(Params...)){
  LuaCallable* callable = new LuaCallable_CppFunction<RetVal, Params...>(func);
  Push(L, callable);
}
```

Now, finally, C++ functions can be exposed to Lua without any extra hassle.

```c++
LuaState L;
L.SetGlobal("average", average);
```

Now, C++ functions can be exposed for use from within Lua.
With trivial modifications, this can be extended to any `std::function`,
  not just C-style function pointers.

Next up, starting to expose C++ classes for use in Lua.
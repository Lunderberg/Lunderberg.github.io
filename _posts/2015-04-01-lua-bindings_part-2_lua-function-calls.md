---
layout: post
title: Lua bindings, Calling Lua Functions
tags: [c++, lua]
---

From [last time]({% post_url 2015-03-31-lua-bindings_part-1_global-variables %}),
  the messiest part of the code was the calling of the Lua function.
It gets even messier when there are arguments being passed or returned.
Suppose we want to call the following Lua function.

```lua
function sum(a, b)
  return a + b
end
```

Using the `LuaState` class made previously, it would be called as follows.

```c++
LuaState L;
L.LoadFile("script.lua");

// Push the function onto the stack
lua_getglobal(L.state, "sum");

// Push the arguments onto the stack
Push(L.state, 5);
Push(L.state, 7);

// Call the function
lua_pcall(L.state, 0, 0, 0);

// Read the result from the stack.
int output = Read<int>(L.state, -1);
lua_pop(L.state, 1);

assert(output == 12);
```

Let's clean it up a bit.
First, implementing a `PushMany()` function, based on the `Push()` functions described before.
Rather than pushing a single value onto the Lua stack,
  it will push every argument onto the Lua stack.
This is possible thanks to variadic templates, a new feature in C++11.
In addition, `std::forward` is used to prevent copying from happening at every step.
This is referred to as "perfect forwarding",
  because no matter what the argument type, it is just passed down to the next function call.

```c++
void PushMany(lua_State* L) { }

template<typename FirstParam, typename... Params>
void PushMany(lua_State* L, FirstParam&& first, Params&&... params){
  Push(L, std::forward<FirstParam>(first));
  PushMany(L, std::forward<Params>(params)...);
}
```

The templated function will push a single value onto the stack,
  then call itself with one fewer parameters.
Once it runs out of parameters, the recursion ends.
With this, we can now make a generic function to call any Lua function,
  passing arguments in.
This is inside the `LuaState` class that was made in the previous post.

```c++
template<typename RetVal = void, typename... Params>
RetVal Call(const char* name, Params&&... params){
  lua_getglobal(name);
  PushMany(std::forward<Params>(params)...);
  lua_pcall(L, sizeof...(params), 1, 0);
  LuaDelayedPop delayed(L, 1);

  return Read<RetVal>(L, -1);
}
```

Note the new class `LuaDelayedPop` as well.
This is a class that will pop a value from the stack in its destructor.
With this, even if `Read()` throws an exception,
  the stack will still be returned to its proper state.

Now, the earlier example becomes much cleaner.

```c++
LuaState L;
L.LoadFile("script.lua");

int output = L.Call<int>("sum", 5, 7);

assert(output == 12);
```

Now that the we can call Lua functions from C++,
  the next step is calling C++ functions from Lua.
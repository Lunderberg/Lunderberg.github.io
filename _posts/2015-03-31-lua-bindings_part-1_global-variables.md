---
layout: post
title: Lua bindings, Global Variables
tags: [c++, lua]
date: 2015-03-31 21:00
---

Yeah, yeah, I know.
Global variables are bad, treacherous, make the program hard to reason about,
  lead to spaghetti code, and smell funny.
However, they are a good first step for passing information
  back and forth between Lua and C++.

First, the Lua code that we want to run.
This is in a separate file named `script.lua`.

```lua
function lua_func()
  x = x + 1
end
```

Each time, it increments a global variable.
Next, the basic C++ code to set up the `lua_State` and define a global variable.

```c++
// Initialize the lua_State
lua_State* L = luaL_newstate();

// Define the global variable 'x'
lua_pushnumber(L, 0);
lua_setglobal("x");

// Load the file
luaL_loadfile(L, "script.lua");
lua_pcall(L, 0, LUA_MULTRET, 0);

// Call the function
lua_getglobal(L, "lua_func");
lua_pcall(L, 0, 0, 0);

// Get the value of 'x'
lua_getglobal(L, "x");
int x = lua_tonumber(L, -1);
lua_pop(L, 1);

assert(x == 1);

// Close the lua_State
lua_close(L);
```

This works, but isn't very clean.
Everything must be done directly.
The state must be manually closed, which could easily be forgotten.
Let's start encapsulating some of this behavior.
Here, `Push(lua_State* L, T value)` and `Read<T>(lua_State* L)`
  are templated function which call the appropriate `lua_push*` function
  or `lua_to*` function for the given type.

```c++
class LuaState{
public:
  LuaState() {
    L = luaL_newstate();
  }

  ~LuaState() {
    lua_close(L);
  }

  void LoadFile(const char* name){
    luaL_loadfile(L, name);
    lua_pcall(L, 0, LUA_MULTRET, 0);
  }

  template<typename T>
  void SetGlobal(const char* name, T value){
    Push(L, value);
    lua_setglobal(L, name);
  }

  template<typename T>
  T CastGlobal(const char* name){
    lua_getglobal(L, name);
    T output = Read<T>(L, -1);
    lua_pop(L, 1);
    return output;
  }

  lua_State* L
};
```

Note that the held `lua_State*` is currently public.
As the implementation becomes more complete, this will become a private variable.

Now, with this, the usage becomes much simpler.

```c++
LuaState L;
L.SetGlobal("x", 0);
L.LoadFile("script.lua");

// Call the function
lua_getglobal(L.state, "lua_func");
lua_pcall(L.state, 0, 0, 0);

// Get the value of 'x'
int x = L.CastGlobal<int>("x");

assert(x == 1);
```

That function call is still rather ugly.
We'll be focusing on that in the next post.
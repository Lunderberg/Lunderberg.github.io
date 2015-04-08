---
layout: post
title: Lua bindings, C++ Functions, Loose Ends
tags: [c++, lua]
---

There are a few loose ends that are left over from the
   [exposing of C++ functions]({% post_url 2015-04-02-lua-bindings_part-3_cpp-function-calls %})
   to be used in Lua.

First, the current implementation leaks memory pretty badly.
Every time a function is exposed to Lua, it constructs a new `LuaCallable_CppFunction` on the heap,
  which is never deleted.
To solve this, we take advantage of Lua's garbage collection.
Just as a metatable can have a `__call` method to call userdata,
  it can also have a `__gc` method to state what should happen when userdata is deleted.
Here, I will make a second function for the purpose of deleting the `LuaCallable` once it is done.

```c++
int garbage_collect_lua_callable(lua_State* L){
  void* storage = lua_touserdata(L, 1);
  LuaCallable* callable = *static_cast<LuaCallable**>(storage);
  delete callable;
  return 0;
}
```

Now, modifying the `Push()` function

```c++
void Push(lua_State* L, LuaCallable* callable){
  void* userdata = lua_newuserdata(L, sizeof(callable));
  *static_cast<LuaCallable**>(userdata) = callable;

  int metatable_uninitialized = luaL_newmetatable(L, "lua-bindings_cpp-function");
  if(metatable_uninitialized){
    Push(L, call_lua_callable);
    lua_setfield(L, -2, "__call");

    Push(L, garbage_collect_lua_callable);
    lua_setfield(L, -2, "__gc");

    Push(L, "Access restricted");
    lua_setfield(L, -2, "__metatable");
  }
  lua_setmetatable(L, -2);
}
```

Now, the `garbage_collect_lua_callable()` function will be called
  whenever Lua's garbage collection decides that the function is no longer needed.
In addition, by assigning a value to the `__metatable` field,
  we make the metatable itself be inaccessible from within Lua.
This is a safety feature, as otherwise, someone might manually call the garbage collection
  causing multiple deletes to be called.


Second, as I alluded to earlier, there is some template magic in `indices` and `build_indices`.

```c++
template<int... Is>
struct indices {};

template<int N, int... Is>
struct build_indices
  : build_indices<N-1, N-1, Is...> {}

template<int... Is>
struct build_indices<0, Is...> : indices<Is...> {};
```

This trick is taken from [loungecpp.wikidot.com](http://loungecpp.wikidot.com/tips-and-tricks%3aindices).
It recursively counts down, generating each index.
A function accepts an argument of type `indices<Indices...>`,
  and is passed an argument of type `build_indices<N>`.
This then generates all indices from `0` to `N-1`.

This is needed so that, at compile time, many arguments can be read from the Lua stack.
Consider the following code.

```c++
void function(){
  func(Read<double>(L, 1), Read<int>(L, 2), Read<std::string>(L, 3) );
}
```

With the indices trick, we can make this into a variadic function as follows.

```c++
template<int... Indices, typename... Params>
void function_helper(indices<Indices...>){
  func(Read<Params>(L, Indices+1)...);
}

template<typename... Params>
void function(){
  function_helper<Params...>(build_indices<sizeof...(Params)>());
}
```

By passing `build_indices` into the helper function, we fix the value of `Indices`.
This then allows us to make the appropriate function call each time to `Read()`.
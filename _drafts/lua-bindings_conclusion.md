---
layout: post
title: Lua bindings, Conclusion
tags: [c++, lua]
---

Now that we have everything set up, looking at the final product.

```c++
class TestClass{
public:
  TestClass() : x(0) { }
  TestClass(int x) : x(x) { }

  int GetX() { return x; }
  void SetX(int x_new;) { x = x_new; }

private:
  int x;
};

Lua::LuaState L;
L.LoadSafeLibs();
L.MakeClass<TestClass>("TestClass")
  .AddConstructor<>("TestClass_default")
  .AddConstructor<int>("TestClass")
  .AddMethod("GetX", &TestClass::GetX)
  .AddMethod("SetX", &TestClass::SetX);

TestClass var = L.Call<TestClass>("lua_func");
```

In order to call the following code in Lua.

```lua
function lua_func()
    var = TestClass_default()
    var:SetX(5)
    return var
end
```

The best part is that this now works around my existing classes in other projects.
If I want to add bindings to Lua, I don't need to modify existing code.

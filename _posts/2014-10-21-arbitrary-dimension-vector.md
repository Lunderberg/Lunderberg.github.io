---
layout: post
title: C++ Geometric Vector in n dimensions
tags: [c++]
---

Geometric vectors represent a point or a displacement from a point.
These can make spatial

The most common use of templates in C++ is to accept an arbitrary class.
However, they can also be used to define value-type parameters.
Here, using a template parameter to define N, the number of dimensions of the vector.

```c++
#include <array>
template<unsigned int N>
struct GVector{
  std::array<double,N> data;

  GVector(std::array<double,N> arr) : data(arr) {}
};
```

From here, one can define all of the standard operators for adding, subtraction, and scalar multiplication.
However, it isn't the most convenient to use.
Whenever the calling code wants to access the internal representation,
  it must be done through the `data` member.
We could define accessors `X()`, `Y()`, and `Z()` for the common usages.
If we make these accessors, we would want them to be available only when it is sensible.

C++11 introduces `static_assert`.
This is used to make a function valid only when certain conditions
The following code then adds accessors that exist only when the dimension is present.

```c++
double& X() {
  static_assert(N>=1,"X available only for dimensions >= 1");
  return data[0];
}

double& Y() {
  static_assert(N>=2,"Y available only for dimensions >= 2");
  return data[1];
}

double& Z() {
  static_assert(N>=3,"Z available only for dimensions >= 3");
  return data[2];
}
```

Note that the check happens after a method is chosen, not before.
Therefore, this cannot be used to make two different methods,
each of which gets called according to the dimension.
To do that, we would need to use `std::enable_if` instead.
Though no such methods are used in this vector class,
it could be used as follows.

```c++
template<unsigned int T = N>
typename std::enable_if< T==1, int >::type
magic_func(){return 1;}

template<unsigned int T = N>
typename std::enable_if< T!=1, int >::type
magic_func(){return 2;}
```

The inner template is necessary to prevent the method from being determined when the class is determined.
This would cause the entire class to be labelled as invalid.
Instead, we only want the function to be labelled as invalid,
and so we need to use a dummy template parameter to delay the method's determination until call-time.

Back on the vector class,
so long as we are using C++11 features, lets make the constructors be more convenient.
So far, any calling code must first contruct a `std::array` before creating a `GVector`.
It would be much more convenient if we could simply construct it as `GVector<2> gv(1,2);`.
In addition, this constructor should be available only with the correct number of arguments.

To accomplish this, we use variadic templates, introduced in C++11.
The `static_assert` ensures that identically N parameters have been passed to the constructor.

```c++
template<typename... Args>
GVector(Args... args) : data{double(args)...} {
  static_assert(sizeof...(args) == N,
                "Arguments passed do not match templated size");
}
```

This class is available in its current form
 on github at [my current project](http://github.com/Lunderberg/omnicolor-images/blob/master/include/GVector.hh).
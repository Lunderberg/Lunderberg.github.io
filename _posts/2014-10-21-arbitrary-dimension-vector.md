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

C++11 introduces `std::enable_if`.
This is used to conditionally include templated code when some value is present.
The following code then adds accessors that exist only when the dimension is present.

```c++
template<typename = typename std::enable_if< N>=1 >::type>
double& X(){return data[0];}
template<typename = typename std::enable_if< N>=1 >::type>
const double& X() const {return data[0];}

template<typename = typename std::enable_if< N>=2 >::type>
double& Y(){return data[1];}
template<typename = typename std::enable_if< N>=2 >::type>
const double& Y() const {return data[1];}

template<typename = typename std::enable_if< N>=3 >::type>
double& Z(){return data[2];}
template<typename = typename std::enable_if< N>=3 >::type>
const double& Z() const {return data[2];}
```

So long as we are using C++11 features, lets make the constructors be more convenient.
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
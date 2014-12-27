---
layout: post
title: Runge-Kutta in C++
tags: [math, c++]
---

In my [previous post]({% post_url 2014-12-23-runge-kutta %}),
  I went through the benefits of using Runge-Kutta for numerical solving of differential equations.
Now, a sample implementation.

Here, `F` is some type that supports addition and scalar multiplication.
In mathematics terms, `F` is a vector space.
The simplest form could be a `float` or a `double`.
For more complicated differential equations, `F` could be a three-dimensional point, for example.

`D` is some function or functor that can be called, returning the derivative of the system.
Rather than using a second template parameter, this could be replaced by either
  `std::function<F(const F&,double)> derivative` or
  `F (*derivative)(const F&,double)`.
However, the first introduces some overhead, and the second is less general.
Since I am already templating on `F`, I might as well template on `D` as well.

```c++
template<typename F, typename D>
class RungeKutta{
public:
	RungeKutta(F initial, D derivative,
	           double deltaT, double start_time=0){
		m_current = initial;
		m_der = derivative;
		m_deltaT = deltaT;
		m_time = start_time;
	}

	void Step(double dT){
		F k1 = m_der(m_current, m_time)*dT;
		F k2 = m_der(m_current + 0.5*k1, m_time + 0.5*dT)*dT;
		F k3 = m_der(m_current + 0.5*k2, m_time + 0.5*dT)*dT;
		F k4 = m_der(m_current + k3, m_time + dT)*dT;
		m_current = m_current + (k1 + k2*2 + k3*2 + k4) * (1.0/6.0);
		m_time += dT;
	}

	void Step(){ Step(m_deltaT); }
	F GetCurrent(){ return m_current; }
	double GetTime(){ return m_time; }

private:
	F m_current;
	D m_der;
	double m_deltaT;
	double m_time;
};
```

This assumes that the following functions are available for `F` and `D`.

```c++
F operator+ (const F&, const F&);
F operator* (const F&, double);
class D{
	F operator() (const F&, double);
};
```
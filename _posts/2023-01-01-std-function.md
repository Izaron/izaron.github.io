---
title: The simplest std::function explanation in 15 minutes
date: 2023-01-01 19:15:00
tags: [story]
categories: [Articles]
---

This post was written under influence of a cool video from Jason Turner *"A simplified implementation of std::function"*

{% include embed/youtube.html id='xJSKk_q25oQ' %}

Often people don't think about how `std::function` works.
Most people know that this thing is a wrapper over *something* that can be *"called"* as a function.
Some people vaguely remember that `std::function` somehow mess with memory on the heap.
The [cppreference](https://en.cppreference.com/w/cpp/utility/functional/function) does not reveal much about internals of its implementation.

We can say that there are two types of objects in C++ on which have the semantics of "calling as a function".
We can name them `Callable`. These two types are:
1. Functions itself:
```c++
int foo(int a, int b) { return a + b; }
```
2. Objects of types that have `operator()`, often they called "functors":
```c++
struct foo {
    int operator()(int a, int b) { return a + b; }
};
```

All other `Callable`s are derived from these two types.
In particular, lambdas is the second type: the compiler converts them into structures with `operator()`.
I once reviewed a [good book about lambdas](/posts/lambda-story) in this blog.

So, a `std::function<Signature>` object is supposed to be able to hold any `Callable` with the given `Signature`.
```c++
template<typename Ret, typename... Param>
class function<Ret(Param...)> {
    // implementation
};
```

There is a problem - a `std::function` type should have a fixed size,
but a `Callable` of type can have an unknown size.
For example, the size of a lambda structure depends on which [captures](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture) it does.

Therefore, unfortunately, a `std::function` object stores the `Callable` in the heap.

An implementation also needs to use tricks like using a virtual class, which will calculate the address of the function to call
independently for each individual type.
They would hold a pointer to this class:
```c++
    struct callable_interface {
        virtual Ret call(Param...) = 0;
        virtual ~callable_interface() = default;
    };
    std::unique_ptr<callable_interface> callable_ptr;
```
Its derived class for a given `Callable` holds a `Callable` object itself and overrides the method to call the right function:
```c++
    template<typename Callable>
    struct callable_impl : callable_interface {
        callable_impl(Callable callable_) : callable{std::move(callable_)} {}
        Ret call(Param... param) override { return std::invoke(callable, param...); };
        Callable callable;
    }
```

A `std::function` constructor takes a `Callable` and makes an object in the heap:
```c++
    template<typename Callable>
    function(Callable callable)
        : callable_ptr{std::make_unique<callable_impl<Callable>>(std::move(callable))}
    {}
```

Finally, calling the `opetator()` dispatches to the right place:
```c++
    Ret operator()(Param... param) { return callable_ptr->call(param...); }
```

This is how *type erasure* in C++ can look like 👍

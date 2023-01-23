---
title: 📚 Book review&#58; "C++ Lambda Story"
date: 2022-08-01 16:39:00
tags: [books]
---

![](/assets/img/posts/2022-08-01/cover.png){: .w-25 .left}

[Link to Leanpub](https://leanpub.com/cpplambda).

As you know, the C++ language is very simple, in just 157 pages you can understand how lambdas work in C++ ✨ (pun intended)

I'd recommend to watch a video joke before reading the book - [C++ Lambda Ace Attorney](https://www.youtube.com/watch?v=T1mTQitLd78) 😃

The book contains an investigation on the entire field of the feature: how people lived without lambdas before C++11,
and how lambdas evolved from standard to standard (C++14/17/20).

Initially lambdas had a simple idea - this code:
```c++
auto lam = [](double param) { /* do something */ };
```
... should be equivalent to this code:
```c++
struct {
    void operator()(double param) const { /* do something */ }
} lam;
```
... that is, to be a simpler form to define *functors* (objects of class with `operator()`), which were used widely
in C++98/03.

All further changes in the lambda design are related to the general development of the language in order to attach new features to the functor.
The book gives an insight of how lambdas look "inside", so many of the question become self-evident:
- 🤔 Why capture is needed only for auto variables;
- 🤔 How to capture `this`;
- 🤔 What is a serious difference between lambdas that do not capture anything, and those that do it; and much more...

I already knew a lot in advance (I even looked into the part of a compiler's source code that is responsible for lambdas),
so I was wondering if there are interesting facts unknown to me? It turned out that there are:
- 🚀 Starting from C++20, it is possible to create a lambda object (previously it was not possible):
```c++
auto foo = [](int a, int b) { return a + b; };
decltype(foo) bar;
// ^ (until C++20) error: no matching constructor for initialization of 'decltype(fo
```
It is needed to pass the lambda's type to whatever needing it, for example as a `std::set` template parameter - [link to godbolt](https://godbolt.org/z/f11xeY8xd).
- 🚀 Starting from C++17, it is possible to use [std::invoke](https://en.cppreference.com/w/cpp/utility/functional/invoke) to increase readability
when calling a lambda immediately:
```c++
auto v1 = [&]{ /* .../ }();
auto v2 = std::invoke([&]{ /* .../ });
```
[Link to godbolt](https://godbolt.org/z/6hr33nKEf).

- 🚀 If the lambda captures nothing, it can be converted to a function pointer.
If you write `+` before the lambda (which is some object from perspective of the compiler), you'll get a function pointer.
It's the simplest way to get a function pointer without using `static_cast` or kind of this. There is a
[thread on stackoverflow](https://stackoverflow.com/questions/18889028/a-positive-lambda-what-sorcery-is-this).

- 🚀 You can inherit from lambda types in many ways. The resulting type will have a number of `operator()(...)`
(with different signatures). There are a number of patterns where this is useful, but it looks eerie.
For example, there is a [std::visit](https://en.cppreference.com/w/cpp/utility/variant/visit) pattern:
```c++
std::visit(overloaded{
    [](A a) { std::cout << a.name << std::endl; },
    [](B b) { std::cout << b.type << std::endl; },
    [](C c) { std::cout << c.age << std::endl; }
}, something);
```

The rest of the "tricks" didn't really surprise me: lambdas in the container, lambdas in multithreading,
capture of the `[*this]` object, template lambdas... They looked self-evident, but someone might be interested 🙂

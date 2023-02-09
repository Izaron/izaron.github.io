---
title: How to discard a [[nodiscard]]
date: 2022-12-06 10:17:00
tags: [creepy]
categories: [Articles]
---

The [\[\[nodiscard\]\]](https://en.cppreference.com/w/cpp/language/attributes/nodiscard) attribute was added in C++17.
It is used to mark functions so that whoever called the function does not ignore the return value.

In many environments, any warning breaks compilation (for example, when using the `-Werror` flag). So is it still possible to ignore the returned value?

It turns out possible! They also have [std::ignore](https://en.cppreference.com/w/cpp/utility/tuple/ignore) in C++.
This is an object that can be assigned any value, without effect.

```c++
[[nodiscard]] int status_code() { return -1; }
[[nodiscard]] std::string sample_text() { return "hello world"; }

void foo() {
    std::ignore = status_code(); // no warning/error
    std::ignore = sample_text(); // no warning/error
}
```

But in the chaotic spirit of C++, we can prohibit `std::ignore` for some types. After digging into the implementation of the standard library,
you can do this, for example for the `int` type:
```c++
template<>
const decltype(std::ignore)&
decltype(std::ignore)::operator=(const int&) const = delete;
```

[Link to godbolt](https://godbolt.org/z/6Tn6G3EK3).

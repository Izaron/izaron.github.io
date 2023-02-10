---
title: std::unreachable - safe shooting yourself in the foot 🔫
date: 2022-08-25 11:01:00
tags: [advice]
categories: [Article]
---

They standardized [std::unreachable](https://en.cppreference.com/w/cpp/utility/unreachable)
in C++23. The function has a wacky description:
> invokes undefined behavior

(Prior to C++23 you can use `__builtin_unreachable` on Linux)

There are cases when it is absolutely known that the values of the argument in the given function
are a limited set, but we are still required to do something on "unreachable" values.

Let's suppose that the `magic_func` function accepts only values `1` or `3`:
```c++
int magic_func(int value) {
    switch (value) {
    case 1:
        return 100;
    case 3:
        return 500;
    default:
        /* ???????????? */
    }
}
```

You need to write useless code - what to do when the value is not equal to `1` or `3`. Usually two options are used:
```c++
    return 0; // returning a dummy value
    throw std::exception(); // throwing a dummy exception
```

Extra code generates extra instructions: [link to godbolt](https://godbolt.org/z/ncnsaMoeY).

The `unreachable` instruction has no semantics, and is needed to show the compiler that
this section of code is "unreachable". The compiler can somehow optimize this section of code.

`undefined behavior` means that anything the compiler wants can happen in this section of code.

In our case, if you write `unreachable` ([link to godbolt](https://godbolt.org/z/9qd5s7oeT)),
the compiler will optimize out an extra check and the code will become equivalent to this:
```c++
int magic_func(int value) {
    if (value == 1) {
        return 100;
    }
    return 500;
}
```

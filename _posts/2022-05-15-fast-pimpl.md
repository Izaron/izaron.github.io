---
title: Fast Pimpl
date: 2022-05-15 17:28:00
tags: [madskillz]
categories: [Article]
---

There is the [Pimpl](https://en.cppreference.com/w/cpp/language/pimpl) pattern that is used to remove implementation details of a class by placing them in a separate
class.

Before using the Pimpl, the code may look like this:
```c++
#include <third_party/json.hpp>
struct Value {
    third_party::Json data_;
};
```

A standard approach is to change `T` to `std::unique_ptr<T>`, because it allows to use an incomplete class:
```c++
namespace third_party { struct Json; }
struct Value {
    std::unique_ptr<third_party::Json> data_;
};
```

There's a trade-off: the memory for the `T` object will be allocated on the heap. So using the Pimpl slows down the run-time
performance ⏱ Therefore one may consider using the Fast Pimpl pattern:
```c++
struct Value {
    std::aligned_storage<sizeof(T), alignof(T)> data_;
};
```
(you have to place the correct numbers instead of `sizeof(T)` and `alignof(T)`)

That is, the struct now holds the raw storage for the object of type `T`, but not the object itself.
The struct now is obliged to manage the object's lifetime correctly.

There are some implementations of this pattern on the Internet (you may find them googling "Fast Pimpl"),
I liked [this one](https://github.com/sqjk/pimpl_ptr) the most 🍬

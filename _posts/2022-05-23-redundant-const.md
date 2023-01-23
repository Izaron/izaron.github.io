---
title: Redundant "const" harms performance ⏱
date: 2022-05-23 11:02:00
tags: [advice]
---

The correct using of `const` is a big topic in C++. The coolest explanation of this topic
is probably there: [Const Correctness, C++ FAQ](https://isocpp.org/wiki/faq/const-correctness),
but it doesn't tell about one disadvantage of the "redundant" `const`.

Let's say we have a structure that represents an API object. The object is immutable, so at first glance it makes
sense to declare all field const:
```c++
struct Widget {
    const std::size_t radius;
    const std::vector<Event> events;
    const Element element;
};
```

But this is bad **when** objects of this type are moved, are hold in containers, and so on.
The fact is that `const`-fields are not movable, therefore when we do `Widget w2{std::move(w1)}`
the fields `events` and `element` are *copied*.

([godbolt example](https://godbolt.org/z/qvbE81WPT) with logs)

Const fields are often not necessary -- it is enough to have a const object (or a pointer/reference to a const object):
```c++
void Do(const Widget& widget);
```

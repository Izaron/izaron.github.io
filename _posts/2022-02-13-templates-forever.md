---
title: Use templates in any unclear situation
date: 2022-02-13 15:52:00
tags: [advice]
categories: [Article]
---

There are two problems that often appear in real world and can be solved with the same tool.

Let's say we have a function that takes...

1. A variable of a very long typename, that has to be written by hand or using a typedef.
```c++
using TBazArray = ::google::protobuf::RepeatedPtrField<Namespace::Foo::Bar::Baz>;
bool AllShallFall(const TBazArray& bazArray) { ... };
```
**Advice**: just use a template, then you won't be searching over the repository to find out what type exactly you are supposed to use.
```c++
template<typename T>
bool AllShallFall(const T& bazArray) { ... };
```

2. A function. A common pattern is that the caller sends a lambda, the callee receives a `std::function` (you don't usually use a "function pointer" in such cases, right?)
The `std::function` has a flaw of allocating memory on the heap, and is not necessary at all.
```c++
bool AllShallFall(std::function<int(void)> callback) { ... };
// ...
AllShallFall([]() { return 4; });
```
**Advice**: you can get rid of the unnecessary layer and pass the lambda "as is". The compiler is able to inline and optimize the code.
```c++
template<typename T> AllShallFall(T callback) { ... };
// ...
AllShallFall([]() { return 4; });
```
One has to be careful about copying big objects. A lambda is a *closure type* object, which size in bytes depends on size of the captured data (references/pointers add 8 bytes, objects taken by value add their `sizeof`). The example above always copies the *closure type* object, but it makes no difference given that it has zero size (the lambda captures nothing).

The code above is not usable when there is a capture of a non-copyable object:
```c++
std::unique_ptr i = std::make_unique<int>(3);
auto l = [i = std::move(i)]() { return 4; };
AllShallFall(l);
```

Therefore, to optimize passing a lambda in any circumstances, it's better to use universal references:
```c++
template<typename T> AllShallFall(T&& callback) { ... };
```

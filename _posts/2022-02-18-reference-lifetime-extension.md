---
title: Reference Lifetime Extension
date: 2022-02-18 22:17:00
tags: [creepy]
---

Today's Creepy Rule of the Standard: if you're initializing a const ref (`const T&`) with a "temporary object"
(via *rvalue*), then the temporary object doesn't get destroyed immediately, but rather lives as long
as the const ref lives.

An example in two lines:
```c++
std::string Foo::GetName();
const std::string& name = obj.GetName(); // this is legal and doesn't crash!
```

Probably the most popular use case of this rule is default argument values:
```c++
void foo(const std::string& s = "default_text");
```

This rule is *very* easy to break. Either you call a method on the temporary object (`obj.GetName().data()`),
or there is an implicit type casting, and so on.
[Abseil tip of the week](https://abseil.io/tips/107) has more examples of successes and failures.

In my experience, the presence of this rule provoked a hellish bug:
```c++
class A { ... };
class B : public A { ... };

void foo(const A& a) { ... }
void foo(const B& b) {

    // ...
    foo(static_cast<A>(b));
}
```

I made a wrong cast that was *copying* the object instead of no-op casting to base class type. The right cast
was `static_cast<const A&>(b)`.

The leg was shot when `foo(const A& a)` started to save the reference to `a` in order to use the reference whenever after.
While `foo` was called, everything was fine, the reference was okay, and just after `foo` has ended, the reference became
dandling. Debugging problems caused by this bug has taken a plenty of my time...

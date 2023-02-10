---
title: Exceptions in C++ and Garbage Collection - what do they have in common? ♻️
date: 2023-01-26 11:59:00
tags: [story]
categories: [Article]
---

I do not mean technical similarity,
but the very principle of having a feature of this sort.
Initially, the feature looks cool and can be used seamlessly "out of the box".

But when there is a need to fix something in this feature,
the programmer will be tortured, digging into the technical implementation of the feature. 🔍

For example, in Java (and other languages with garbage collection),
it is necessary to understand garbage collection algorithms (mark-and-sweep, etc.).
This way the programmers learns how to work more accurate with memory so that the collector collects less garbage (with using things like "object pool"),
and understands the necessary settings so that the collector does not freeze the program in the middle of processing an HTTP request, and so on.
It's all longly and tediously described in various articles.

There is a similar story with exceptions in C++.
But it requires even more non-trivial knowledge for a complete understanding.
There are two main problems, the first one is more common, and the second one is tougher:

1. Runtime overhead and damaged compiler optimizations.
There is a super cool longread: [C++ Exceptions through the prism of compiler optimizations](https://habr.com/ru/company/jugru/blog/494986/) (unfortunately in Russian only,
but the online translator will do fine).

Instead of just calling a function that theoretically can throw an exception,
you have to make an additional block of code to handle a potential exception, and transfer execution there in case of an exception (so you make an extra check for EACH call).
Also, exceptions make harm to many optimizations.
For example, the "dead code elimination" optimization believes that the code for exception handling is always needed,
and there, by transitivity, all variables inside it are also needed, and in result, everything is optimized more poor.

2. Complexity in development, including need for the **Strong Exception Guarantee**.

The Strong Exception Guarantee is a guarantee that if you call a function that ends up suddenly throwing an exception,
the state of the program should remain the same as it was before the function was called.
The STL containers implement SEG whenever possible, and this is very tedious.
In [my article about containers](https://habr.com/ru/post/664044/) I described how the SEG works.
```c++
void TryAddWidget(std::vector<Widget>& widgets) {
    // running some code...
    Widget w;
    try {
        widgets.push_back(std::move(w));
    } catch (...) {
        // caught an exception...
        // we expect that `widgets` remains as it was before calling `push_back`
    }
}
```
C++ has helper functions specifically for the SEG: for example [std::move_if_noexcept](https://en.cppreference.com/w/cpp/utility/move_if_noexcept).

In ordinary programs, it is very difficult to make a normal SEG.
It does not work that way that we accepted the request, put data from it into a structure, wrote something to a database,
wrote logs, and then suddenly caught an exception and we need to return everything back.

There is a concept of the Basic Exception Guarantee, that is, the program continues working,
but its data becomes a little broken.
For example, in the example above, depending on the properties of the `Widget` class, all elements in the `widgets` vector
may be erased after throwing an exception 😐

There is a nice talk about error handling in C++:

{% include embed/youtube.html id='qQPLjxZ56QA' %}

There is a hilarous example of the BEG in the video, I'll copy-past it here:
> Customer: "Hello, it is Microsoft Word support? I was writing a book. Suddenly, Word deleted everything"

> Microsoft: "Oh, that's ok. Word only provides a basic exception guarantee."

> Customer: "Oh, alright then, thank you very much and have a good day!"

In many code styles (for example in [Google C++ Code Style](https://google.github.io/styleguide/cppguide.html)) exceptions are forbidden.

Compilers have the `-fno-exceptions` flag, which removes all the overhead associated with exceptions,
and does not generate unnecessary code. Exceptions can still be thrown, but they will simply crash the program, without processing in any way.

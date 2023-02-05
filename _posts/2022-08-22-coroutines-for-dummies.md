---
title: Coroutines for dummies (like me)
date: 2022-08-22 11:01:00
tags: [story]
---

When I tried to understand how the coroutines (added to C++20) work, I discovered a very small number of understandable explanations on the Internet.

In most explanations, the authors begin to remember in which year the term "coroutine" first appeared,
or they involve goroutines (from the Go language) or Boost.Fiber,
or they kind of calculating how many quintillion times coroutines are faster than threads... 😑

I found a basic understandable **theory** about coroutines here:
[Coroutine Theory](https://lewissbaker.github.io/2017/09/25/coroutine-theory).
It became immediately clear that a coroutine is a function that can suspend and continue (from the moment it has stopped previously)
its execution
[example on godbolt](https://godbolt.org/z/WGncnEvvT):
```c++
TGenerator<int> fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        a = std::exchange(b, a + b);
    }
}
```

So, you have kind of a function `fibonacci()` that can suspend and later continue.
The memory for `a` and `b` doesn't appear out of nowhere (the stack memory is not used),
therefore the compiler would allocate some heap memory for coroutine's local variables.

You'll learn two main facts:
- ✍️ Coroutines is syntactic sugar (just like lambdas), they could improve code readability.
- ✍️ Coroutines as "suspendable functions" are orthogonal to multithreading. Coroutines don't
replace them.

Now it's time to find out the **tasks that are solved by coroutines** better than by other means.
You can use http servers as an example, but frankly this is too unobvious an example.

In my opinion, one of the best posts about usable coroutines:
[How to implement action sequences and cutscenes](https://eliasdaler.github.io/how-to-implement-action-sequences-and-cutscenes/)
about complex logic in games.

It's not in C++, it's in Lua (a simple scripting language), but it doesn't change the concept (and the coroutines look easier there).

C++, as usual, went the maximal hardcore way.
It defines only interfaces for coroutines, and the programmer must write event loops and classes for the awaiter/promise **himself**.

[You'll see a toy event loop here](https://dev.to/atimin/the-simplest-example-of-coroutines-in-c20-4l7a).
There is nothing to do without specialization in this area -
you are to use ready-made libraries, for example [lewissbaker/cppcoro](https://github.com/lewissbaker/cppcoro) or
[YACLib](https://github.com/YACLib/YACLib).

Coroutines are not a bad thing, but the approach to implementation in C++ (with the encouragement of writing everything from scratch)
surprised me. But the overall impression (with the experience of coroutines in Lua/Python)... Just look at some guy's comment, it says better:

> Damn, I used coroutines back in 1987, on ancient machines with 32 KB of memory.
I used it easily and naturally.
The description of the coroutines in that language (SPL — System Programming Language) took up a maximum of one and a half pages
of the manual and was understood even by cooks. After the article about coroutines in C++, the feeling is: "coroutines? — never!".

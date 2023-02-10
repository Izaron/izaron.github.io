---
title: Need for speed!
date: 2022-02-10 20:10:00
tags: [video]
math: true
categories: [Article]
---

When comparing C++ with other popular languages (Python, Java, C#, etc.), it comes clear that adequately written
programs on C++ almost surely will be faster than the analogous programs on the other languages.

But ever C++ is divisible by speed.

1. In standard projects it's common to use classes like `std::shared_ptr` instead of raw pointers, to have a lot of memory
allocations, to forget using `std::move`, etc.
2. It a-bit-non-standard projects (browsers, compilers) things are getting tougher - there is [small vector](https://github.com/facebook/folly/blob/main/folly/docs/small_vector.md) (a vector on the stack), static polymorphism (using [CRTP](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern#Static_polymorphism) instead of virtual methods), `std::string_view` pointing to somewhere instead of `std::string`.
This isn't unusual for most programmers.
3. But real-time programming is another world. It's forbidden to make syscalls, to block threads, to use algorithms
slower than $O(1)$, and so on. This is needed for processing signals and sound, HFT systems...

There is a nice talk about the third disivion by Timur Doumler:

{% include embed/youtube.html id='Tof5pRedskI' %}

This talk was very interesting for me, as I had not so much experience in real-time systems.

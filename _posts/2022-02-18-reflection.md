---
title: Reflection in C++
date: 2022-02-18 12:50:00
tags: [video]
categories: [Articles]
---

One of the problems in C++ is lack of *reflection*. Reflection is the ability of the program to "understand"
its structure. For example - getting the class name, list of methods, adding the methods dynamically, and so on.

C++ will never have a *run-time* reflection, because ~95% of source code's information just disappear after compilation,
therefore there can't be a complete run-time reflection.

Evolution of a *compile-time* reflection was delayed because of imperfection of compile-time computations at all. Therefore
such feature might make it the language not earlier than C++26.

There is a nice talk by **Andrew Sutton** who develops the C++ reflection:

{% include embed/youtube.html id='60ECEc-URP8' %}

Interestingly, this is the second approach to reflection support (constexpr-based). There were the first approach (template-based).

I have a longread about reflection in C++Next based on this video and other sources. You may want to read it, if this topic
is interesting to you 🙂
[https://habr.com/ru/post/598981/](https://habr.com/ru/post/598981/)

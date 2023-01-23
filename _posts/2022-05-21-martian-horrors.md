---
title: Martian Horrors 👽
date: 2022-05-21 21:44:00
tags: [video]
---

As a programmer, I often deal with "air castles": I write code that works with very abstract concepts.

Suddenly I came across a video, where they analyze source code of software for NASA rovers ([nasa/fprime](https://github.com/nasa/fprime)).

{% include embed/youtube.html id='RbhufLudVsI' %}

The first 15 minutes of the video is a dull overview of CI and inhouse .bash-scripts used for builds.
There we learn that according to the evaluation of an authoritative [lgtm](https://lgtm.com/) analyzer the repository contains top-quality `C/C++` code, marked as `A+`.

But then we see a vigorous mix of C89 and C++11 and some great examples of code smell. There you can see:

- Inhouse awkward string and vector implementations.
- Strange code like this:
```c++
ObjBase::~ObjBase() {} // why not `~ObjBase() = default`?
void resetSer(void);
ExternalSerializeBuffer(ExternalSerializeBuffer& other);
ExternalSerializeBuffer(ExternalSerializeBuffer* other);
const char* operator=(const char* src);
(void)snprintf(buffer, size, ...
```
- `reinterpret_cast` used in 169 places.
- Constants are declared via `enum`s.

Comments on the video are more interesting than the video itself -- employees of companies like the European Space Agency
confirm the awkward level of programming in "scientific" and "state" spheres.

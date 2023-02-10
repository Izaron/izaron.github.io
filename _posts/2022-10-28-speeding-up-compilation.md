---
title: Improving C++ compilation speed
date: 2022-10-28 10:01:00
tags: [story]
math: true
categories: [Article]
---

The very recurring topic of "speeding up compilation" is a special fun activity in which many C++
developers who do not like slow compilation took part. I will list the main directions of this activity.

The main cause of problems is the "$N\*M$ problem", almost all approaches try to resolve it.
This means that if there are $N$ header file and $M$ non-header files in the project,
then the total number of files read during compilation will tend to $N\*M$,
because each non-header file is compiled separately and transitively connects almost all headers, if you do not prevent this in advance.

The total size of the file after including all headers can reach hundreds of thousands of LOC.

This is relevant not only when building the project from scratch.
If you change some important header that is included almost everywhere,
then it will be almost equivalent to building the project from scratch.
In worst cases, a change in almost any header triggers a rebuild from scratch.

Ideally, we should strive for $N+M$, as is in many programming languages.

### PImpl

PImpl is a well-known idiom (you can read about it [here](https://marcmutz.wordpress.com/translated-articles/pimp-my-pimpl/)),
whose goal is to hide the API details of the class. But there is a good side effect from using it -
the API's header files may not include some other headers if they use only a pointer to the class (they could use forward declaration)!

**Cons**:
- An object of the desired class will have to be constructed in the heap.
But there is a witchcraft named [Fast PImpl](/posts/fast-pimpl), which, however, has its disadvantages.
- It is difficult to maintain, struct adherence to the rules is needed.

### Jumbo build
(aka **Unity build**, aka **Single Translation Unit**)

[You can read about it there](https://austinmorlan.com/posts/unity_jumbo_build/).

Unity build is an approach where all files in the module are built as a single sile.
This means that if we have files `"aaa.cpp", "bbb.cpp", ..., "zzz.cpp"`, then an artifical file is created (preferably automatically, by the build system) that connects them all:
```c++
/* all.cpp */
#include "aaa.cpp"
#include "bbb.cpp"
// ...
#include "zzz.cpp"
```
And only `"all.cpp"` file is built.
The point is that if 90%+ of the compilation time is taken not by the file itself, but by the headers that it includes,
then this approach will almost linearly speed up compilation.

**Cons**:
- Names of static variables and function in different files will start to conflict.
- In general, this looks creepy.

### Precompiled headers

[You can read about it there](https://en.wikipedia.org/wiki/Precompiled_header).

If we have some very rarely changing headers, then they can be "precompiled".
Clang does it this way: parses a header (say `test.h`), analyzes and saves some intermediate representation (an AST) to a new file (say `test.h.pch`).
The point is that if there is an include of the `test.h` header somewhere, then the compiler will not parse it from scratch,
but will rather read `test.h.pch`, which supposed to be faster.

**Cons**:
- In my experience, for some reasons this approach doesn't work any noticeable faster.
This approach is controversial in the Internet.

### Include What You Use

[IWYI](https://github.com/include-what-you-use/include-what-you-use) is a tool to remove redundant includes.
There is good documentation.
Sometimes there are bugs related to the fact that it is impossible to mathematically prove the uselessness of an include - they all implicitly affect each other.

### A fast virtual server

You can speed up compilation not only with some code tricks. [There I described](/posts/thin-client) how you can use virtual servers with decent resources for development.

### Distributed build

Large companies with large projects use distributed build server - something open source ([distcc](https://www.distcc.org/))
or even in-house ([a recent article from VK (In Russian but an online translation will do fine)](https://habr.com/ru/company/vk/blog/694536/)).

This is an interesting topic, the article describes what a distributed build looks like.
In addition, in order not to build the same things reduntandly,
you can use a cache - if your colleague has already built some file with given hash and environment, then you will not have to wait
for build, because the corresponding object file will be delivered to your machine by the build server.

### The new `mold` linker

There are no problems with linkers in general, but if you have an executable with debug symbols with size of several GBs,
then it can be linking forever, for 2-3-10 minutes on every build.

A new linker named [mold](https://github.com/rui314/mold) is currently being developed,
which is faster than other linkers, but there is no clear documentation about it and some people complain about instability and broken executables.

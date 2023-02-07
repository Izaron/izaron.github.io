---
title: ABI&#58; the three scary letters 🎃
date: 2022-11-25 19:48:00
tags: [creepy]
---

One of the topics that is hardly noticeable in the world outside C++, but which causes a regular shitstorm among senior C++ contributors is the question of **breaking the ABI**.

ABI ([wiki](https://en.wikipedia.org/wiki/Application_binary_interface)) is a set of conditions that the interface
of a software module (library, operating system) satisfies at the binary level: this includes the calling convention, the size of data types, and much more.

The simplest example when this is important is when an executable file (`.exe` on Windows)
requires dynamic libraries ([.dll](https://en.wikipedia.org/wiki/Dynamic-link_library) on Windows or `.so` on Unix).

The executable and the dynamic library are two different programs that should always be compatible with each other.
This means that the executable and the library should be able to update independently of each other so that they can continue to work together when using an updated
version of one of these two programs.

(You can read about the dynamic libraries in [this article](https://amir.rachum.com/blog/2016/09/17/shared-libraries/)
or in [this book](/posts/advanced-compiling))

The problem is that the ABI **is very easy to break** -
there is an [incomplete list of breaking changes](https://www.acodersjourney.com/20-abi-breaking-changes/). Some of the simple examples:
- 🔍 *Adding a new field to a public class*: In general breaks everything,
because now the `.exe` file (which is not recompiled!) has incorrect stack offsets, sizes of dependent classes, and so on.
- 🔍 *Add a new virtual method*: The `.exe` file now incorrectly calculates the addresses of the old virtual methods.

There are tools that check ABI compatibility: [abidiff](https://manpages.ubuntu.com/manpages/xenial/en/man1/abidiff.1.html).

The C++ standard is being developed with focusing on not breaking the ABI in new versions of the standard -
that is, recompiling the project on a newer standard shouldn't break communication with the dependent `.dll`/`.so` libraries.

This leads to problems described in the articles:
- 🔍 [The Day The Standard Library Died](https://cor3ntin.github.io/posts/abi/) - a longread.
- 🔍 [Digital Demons](https://thephd.dev/binary-banshees-digital-demons-abi-c-c++-help-me-god-please) - a harder longread.

There are two classes of problems related to ABI compatibility:
1. You can't change the language internals, for example, sort the class fields so that the class takes up less space (you know, to suffer less from field alignments).
2. It is impossible to actively change the standard library - most proposals are destroyed immediatelly (this class of problems is more harsh).

The implementation of the C++ standard library is not part of the compiler.
The C++ standard only defines the interface of the standard library, and there are [several implementations](https://en.wikipedia.org/wiki/C%2B%2B_Standard_Library#Implementations),
and they all are provided as a **dynamic library**.

Because of this, the C++ standard library is extremely poor,
its containers are slower than necessary, some classes are super akward (`std::initializer_list`, `std::regex`),
and so on - and no one can fix it. If you design something new (for example, a JSON parser), then it will not be possible to change it normally.

Even the designs of Win32, or POSIX, or the Ethernet protocol leaves room for all sorts of changes in the future - but not the standard C++ library.

Every year, the cost of NOT breaking the ABI (that is, the stagnation of the language)
is gradually approaching the cost of breaking the ABI. It is unclear whether it will be allowed to break the ABI, since at least two vendors are against it:
1. GCC. All proprietary products shipped in binary form for Linux rely on
ABI compatibility, so they don't need to rebuild their product for each new distribution.
2. Microsoft. Visual Studio made its ABI forward compatible from 15th version. Libraries compiled in MSVS 2015 are linkable with these linked in 2017 and 2019 version.

Some people think that C++ "preserves compatibility" and this is kind of cool.
In fact, as we can see, the reasons for maintaining compatibility are just business - so people don't need to recompile the product a lot of times for different environments.

After all, if you think logically, nothing good will happen if you try to use some C++ library compiled ~20 years ago.
It will most likely either be built for an old architecture, or using an
[incompatible compiler](https://en.wikipedia.org/wiki/Name_mangling#How_different_compilers_mangle_the_same_functions), or extremely slow
(compilers are getting faster every year).

Everyone has its own opinion about ABI.
I believe that if a commercial product is actively developing and wants to be modern enough,
then one way or another it will use the new features of the language and library.
And in this case, it will fail to work on extremely old distributions with old versions of the standard library, even with the ABI unchanged.

Programming with ABI compatibility in mind is not the most common business scheme.
For example, [Qt](https://www.qt.io/) breaks its ABI every release, and no one has died from it yet.

From my experience, I can say that I have never encountered ABI issues in my entire life -
FAANG-like companies use [monorepos](https://medium.com/geekculture/monorepo-google-meta-twitter-uber-airbnb-and-you-1723db84d301).

These are huge repositories, and all binaries are built statically from sources (without linking dynamic libraries).
It's fast because it uses distributed build servers with caching.

If some common code changes, then all dependent projects and modules in the company will start using it immediately and
CI will also run their tests - so this is an ideal scheme for large companies.

As for the poor old standard library,
a lot of projects and companies use analogues of it, often self-written:
[eastl](https://github.com/electronicarts/EASTL),
[folly](https://github.com/facebook/folly),
[abseil](https://abseil.io/),
[poco](https://pocoproject.org/) and a number of others.

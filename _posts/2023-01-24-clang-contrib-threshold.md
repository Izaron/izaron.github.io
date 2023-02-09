---
title: Entry threshold for contributing to compilers
date: 2023-01-24 19:15:00
tags: [story, compiler]
---

Once in a C++ chat, a guy asked whether it make sense to investigate and send bugfixes to existing compilers?
There definitely must be a lot of people who want to fix bugs, and the issues just disappear instantly, he thought 😱

In fact, the situation with Clang/LLVM (and with many other projects) is the opposite.

There are thousands of issues that no one fixes for years.
The review process is also quite slow,
pull requests may hang for weeks or months.

As I see it, there is a maximum of several dozens active developers, yet a handful of super-active developers.

Many of the mentioned active developers develop the compiler at work,
that is, they work at Google/Apple/%company\_name% and these companies pay them to develop Clang/LLVM.
If no one paid, the development probably has been moving a lot slower than now.

In addition to fixing bugs, anybody can go to the [C++ status page](https://clang.llvm.org/cxx_status.html#cxx20),
look at which C++20/23 features are not implemented in Clang yet and implement something.
I've done this a couple of times.
But these were simple commits.
It is almost impossible to implement complex features, it requires work on scale of a full-time job. Therefore, it's not an easy thing to do.

For example, it was relatively easy to make the C++23 feature
[\[P2324R2] Labels at the end of compound statements](https://wg21.link/P2324R2).
It took me 2 commits - [1st commit](https://reviews.llvm.org/rG510383626fe146e49ae5fa036638e543ce71e5d9),
[2nd commit](https://reviews.llvm.org/rG3285f9a2392fd6bc79241b1e97b124079553e48d).

Also, some progress was made on the
[\[P0533R9] constexpr for \<cmath\> and \<cstdlib\>](https://wg21.link/P0533R9) proposal.
In this proposal, a bunch of new methods are labeled `constexpr`.
I came up with an implementation scheme - a method should be conditionally marked `constexpr` if its constant computation is supported by the compiler.
A simplified example for [std::max](https://en.cppreference.com/w/cpp/numeric/math/fmax):
```c++
inline _LIBCPP_CONSTEXPR_CXX23_IF_CONSTEXPR_BUILTIN(__builtin_fmax) double fmax(double __x, double __y) {
  return __builtin_fmax(__x, __y);
}
```

The `_LIBCPP..._BUILTIN` macro refers to the `__has_constexpr_builtin` function
(introduced in [my commit](https://reviews.llvm.org/rG5b567637e22bfa128514a5a9de7f3296423e8acd))
to check whether the function in its argument is supported in constant computation.

A `__builtin_XXX` builtin is converted into a regular function call or some code in runtime, or executed directly in the compiler in constant computation.
For example, [my commit for supporting the fmax builtin](https://reviews.llvm.org/rG0edff6faa26664772c41fed8d7759bba703f4987).

I also added a test for monitoring implementation progress of `P0533`
([my commit](https://reviews.llvm.org/rGcc2cf8b22b35a9cf87a61bf66c259455185fa6e3)).
As soon as some method is marked `constexpr` (automatically, as described above),
it is necessary to change the macro in the test (otherwise the test will fall).
As soon as all the necessary methods are marked `constexpr`,
the test will say that the `P0533` proposal is fully implemented (and you need to change the status on the `cxx_status.html`).

However, a few months later, it progressed only slightly -
[look at the blame](https://github.com/llvm/llvm-project/blame/c3728d28821e212bd3658261e58e744421668720/libcxx/test/libcxx/numerics/c.math/constexpr-cxx2b-clang.pass.cpp),
there were only two commits. A guy who develops libcxx, supported some of the functions.

Therefore, anyone can contribute to compilers if they have free time and desire to do something new.

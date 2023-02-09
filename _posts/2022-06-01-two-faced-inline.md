---
title: Two-faced "inline" for function
date: 2022-06-01 11:58:00
tags: [creepy]
categories: [Articles]
---

So it happened that C++ has the `inline` keyword, which being applied to functions gives them two orthogonal perks.
The one perk appeared because of the other perk, but they're independent otherwise. The said perks are:
1. [\[dcl.inline\].2](https://eel.is/c++draft/dcl.inline#2): A hint to the compiler that it is more preferable
to put the code from the function to the place of the function call, than calling the function itself.
(In the Clang compiler this is the `inlinehint` function attribute, in terms of LLVM IR)
2. [\[dcl.inline\].6](https://eel.is/c++draft/dcl.inline#6): A hint to the compiler that the function might have
more than one *definition* in the program. The programmer should secure that these definitions are the same - this condition
is usually met because the method code is written only once in a header file.
(In the compiler this is the `linkonce_odr` function attribute)

Therefore, the function is given two attributes: `inlinehint` and `linkonce_odr`.

The fun part is that `inlinehint` is given only to functions which are defined with the `inline` keyword.

Clang does exactly this: `inlinehint` appears if and only if there was the `inline` keyword.

Therefore, the function `inline int random() { return 4; }` gets **both** `inlinehint` and `linkonce`.

But functions that are *implicit inline functions* in terms of the Standarts, like these:
- Template instantiations;
- *default* and *deleted* and compiler-generated class methods;
- Class methods which definitions are inside the class definition;
- `constexpr` and `consteval` functions (actually only `constexpr`, because `consteval` functions don't get into executable files);

... they get **only** `linkonce`. One has to write the `inline` keyword anyway to make it `linkonce`+`inlinehint`.

I wish *inline functions* were renamed into *linkonce functions*, because the current Standard wording is quite confusing...

## FAQ

**Question:** Does `inlinehint` attribute affect the compiler much? I've heard that compilers have been ignoring these "hints" for a long time...

**Answer:** The effect is practically invisible. Inlining threshold for a regular function is 225 points, for an `inlinehint`-function
is 325 points ([link to the code](https://github.com/llvm/llvm-project/blob/39b8a27132833a7c91059e26af48f20b2e874b61/llvm/lib/Analysis/InlineCost.cpp#L74-L80)),
and it was measured by someone that you'd need ~1000 points to make a real difference.

Therefore, in 99% of cases you'll get the same output for `inline constexpr void foo()` and `constexpr void foo()`. But the clang-tidy maintainers
[reject the patch](https://reviews.llvm.org/D18914)
to check redundant inlines because of the vague 1% of cases.

**Question:** How the attributes depend on each other if they are orthogonal by effect?

**Answer:** To inline the function (the `inlinehint` attribute), the compiler should "see" the definition of this function
somewhere in the translation unit. Usually this is not a good thing, because if we have N translation units, we'll
get N definitions of the same function and the linker has to deal with it. Therefore the function should have the `linkonce`
attribute in order to not break the ODR.

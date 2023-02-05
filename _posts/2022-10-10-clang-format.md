---
title: clang-format&#58; an architectural failure? 🩼
date: 2022-10-10 12:47:00
tags: [compiler]
---

I wrote something about [clang-tidy in this blog](/posts/clang-tidy) previously.
It is used to check the code quality.
It has a modular architecture - everyone can write their own check and use many checks independently from each other.
It works at the [AST level] (https://clang.llvm.org/docs/IntroductionToTheClangAST.html),
that is, the code passes [lexical](https://en.wikipedia.org/wiki/Lexical_analysis) and
[syntactic](https://en.wikipedia.org/wiki/Parsing) analysis before using the tool.

And [clang-format](https://clang.llvm.org/docs/ClangFormatStyleOptions.html) is another tool used for formatting the source code -
so that there are the right number of spaces, sorted `#include` and so on. It works at the token level, that is, the code passes only lexical analysis before using the tool.

That is, the `clang-format` understands only approximately what the current token doest and what is needed to be done.
For example, the code
```c++
class A: B {
```
is seen as this sequence of tokens:
```
(kw_class) (identifier) (colon) (identifier) (l_brace)
```

And `clang-format` applies a series of hardcoded rules on these tokens, with using various ad-hoc stuff like nesting stack for brackets.
There is no modularity at all, that is, all the rules are written right somewhere deep in the source code of tool.

For example, at some point in the middle of its work, the [WhitespaceManager::generateReplacements](https://github.com/llvm/llvm-project/blob/bc839b4b4e27b6e979dd38bcde51436d64bb3699/clang/lib/Format/WhitespaceManager.cpp#L98-L115)
method is called, which corrects spaces, and in it inside the [WhitespaceManager::alignArrayInitializers](https://github.com/llvm/llvm-project/blob/bc839b4b4e27b6e979dd38bcde51436d64bb3699/clang/lib/Format/WhitespaceManager.cpp#L1080-L1102)
method which corrects gaps in arrays.

It is difficult to format tokens without semantics at all,
so `clang-format` "annotates" tokens with additional data before formatting: it maps each `Token` object to some `FormatToken` object.

There is a number of ad-hoc fields like [bool IsArrayInitializer](https://github.com/llvm/llvm-project/blob/874c0327e7e90d42198fd3aba5e3d636e0ba87c3/clang/lib/Format/FormatToken.h#L500)
(indicates whether this token is the beginning of an [array initialization](https://en.cppreference.com/w/c/language/array_initialization)),
or [FormatToken *MatchingParen](https://github.com/llvm/llvm-project/blob/874c0327e7e90d42198fd3aba5e3d636e0ba87c3/clang/lib/Format/FormatToken.h#L485)
(pointer to the closing parenthese).

Everything works notoriously bad with this approach😣. The common error is putting a lot of extra spaces and disfiguring lambdas.

There is [a bunch of issues related to clang-format](https://github.com/llvm/llvm-project/labels/clang-format),
and fixing them is much more difficult than fixing issues related to `clang-tidy`.

If you're fixing a bug in `clean-tidy`, the "area of potential edits" is the code of a separate check (a maximum of several hundred lines),
then in `clang-format` it is the entire code of `clang-format`.

For example, it is very difficult to fix such a stupid bug as broken formatting in the
[array initializer nested inside brackets](https://github.com/llvm/llvm-project/issues/55493#issuecomment-1272639004).
The fact is that formatting relies on the "annotation" of tokens, and it is exactly what it is for nested brackets.
So it is necessary to fix the "annotator", but it is way too difficult and there is a risk of breaking something else.

And so for many issues - you begin to understand a minor problem, like "why an extra gap is being written",
you reveal the whole chain of reasons, and you get a mega-problem that is impossible to fix.

Therefore, try to make modular programs to reduce the "area of potential edits" when fixing a bug 😎

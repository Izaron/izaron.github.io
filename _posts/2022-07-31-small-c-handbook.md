---
title: 📚 Book review&#58; "The Small C handbook"
date: 2022-07-31 16:39:00
tags: [retro, books, compiler]
---

![](/assets/img/posts/2022-07-31/cover-en.jpg){: .w-25 .left}

![](/assets/img/posts/2022-07-31/cover-ru.jpg){: .w-25 .left}

[Link to Amazon](https://www.amazon.com/Small-C-Handbook-James-Hendrix/dp/0835970124).

This book has quite an interesting story.

There are two versions of the book on the photos. The photo on the left represents the original **1984 "The Small C Handbook"**.
The photo on the right is its translation to the Russian language, **1989 "A C compiler for micro-computers"**.

I read this book to learn how much compilers have changed over years.
The book has historical value as a view into the realities of development 30-40 years ago.

The [Intel 8080](https://ru.wikipedia.org/wiki/Intel_8080 ) microprocessor, then widely popular, is used as a "microcomputer".
It has an 8-bit architecture, seven 8-bit registers, and 16-bit memory addressing (which lets addressing up to 64 KB = 65,536 bytes of memory).

In the [Warsaw Pact](https://en.wikipedia.org/wiki/Warsaw_Pact) countries many technologies were copied.
The functional analogue of the Intel 8080 was the [KR580VM80A](https://en.wikipedia.org/wiki/KR580VM80A) microchip,
developed by the Kiev Research Institute of Micro Devices, so Soviet programmers could read books about Intel 8080 and develop something.

A microprocessor is a useful thing, it can be used for computing, traffic lights, printers, slot machines, synthesizers, measuring devices...

The book's "Small C" refers to a subset of the C language that a self-written compiler can compile.
Small C is slightly different from the "full" C language.

All basic constructs are available, but there are serious costraints. For example there are only two types - **char** (1 byte) and **int** (2 bytes).
Intel 8080 in the basic configuration couldn't work with float numbers, one needed a separate [Intel 8231/8232](https://en.wikipedia.org/wiki/Intel_8231/8232 ) coprocessor
for that.

The book consists of several parts.
1. Part 1 describes the 8080 microprocessor, its command system, an overview of its assembler, loaders and program linkers.
The basic things haven't changed in a few decades.
2. Part 2 describes the Small C language, which, in my opinion, is not so different from the "full" C.
3. Part 3 is the most interesting, it describes the compiler and everything related to it, how different constructions
should be translated to the assembler, all sorts of little things (cross-compilation, etc.).

The source code of the compiler is given right in the book - it takes up to half of the book.
I found these sources [on github](https://github.com/nickandrew/smallc/tree/ac9ae5977594395b2241a28282c9ea745e08925f/smallc-21),
it is better to look there (namely files from `cc.def` to `cc42.c`).

What are the features of compilers of that time / of this type:
- 🚀 In just **~3000 lines of code**, one can make his own C compiler from scratch.
And now [LLVM](https://github.com/llvm/llvm-project) consists of at least 11mln lines of C/C++ code.
When a technology is simple, everyone can copy/make it, but with the development of the technology, only a few implementations survive.
C/C++ compilers, web browsers, operating systems - there were dozens of these programs, but only handful of them are actual nowadays.

- 🚀 **The compiler is single-pass** - it parses the file only once from top to bottom. Compilation is very fast.
The C++ compiler can no longer be single-pass.
For example, because of templates or constexpr code.
The simplest example is parsing a class : firstly, the signatures of all methods and fields are parsed, only then the compiler parses the body of the methods.
```c++
class TSomeClass {
public:
    int GetValue() const {
        return Value_; // C++ "sees" Value_, though it is defined "later"
    }
private:
    int Value_;
};
```

- 🚀 **There are no code optimizations**
(except for the most elementary ones like folding the expression `2+3` into the constant `5`).
The book has a chapter on how to write "optimal code", there are strange (from today's perspective) tips like:

🤯 "global variables are better than local ones"

🤯 "it is better to compare expressions with a constant equal to `0`"

🤯 "`++i` is better than `i++`"

- 🚀 Together with the previous paragraph - **there is no intermediate representation of the code**
(like ASTs) -
the compiler generates the assembler in-place.
For example, when parsing an if-expression, labels and conditional branches are created immediately.

- 🚀 There are various limitations with the expectation that the compiler itself runs on a not so powerful machine.
For example, the source code cannot have more than 130 `#define`s in a file,
variable names longer than 9 characters,
more than 200 global variables, and so on...

The compiler's code itself uses global variables too much!

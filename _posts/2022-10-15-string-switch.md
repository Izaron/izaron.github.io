---
title: Simple "switch" for strings 🎲
date: 2022-10-15 17:07:00
tags: [madskillz]
categories: [Article]
---

C++ can't have strings (or string literals) in [switch-expressions](https://en.cppreference.com/w/cpp/language/switch) as `case` values.

There can only be constant integral values or enum values
(an enum itself is an [integer type secretly](https://stackoverflow.com/questions/1122096/what-is-the-underlying-type-of-a-c-enum)).

Such a strict restriction is made for practical reasons -
switch expressions in executable code can be transformed into a super-optimized form using [branch table](https://en.wikipedia.org/wiki/Branch_table),
when the integer value of the argument simply calculates the address of the code to jump to.

It is clear that it is impossible to make branch tables for strings, so the efficiency of such a switch will not differ from a bunch of if-expressions.

In other programming languages, strings in switch are possible: [Java 7](https://docs.oracle.com/javase/7/docs/technotes/guides/language/strings-switch.html),
[C# 6](https://github.com/dotnet/csharpstandard/blob/standard-v6/standard/statements.md#1283-the-switch-statement), but these languages do not focus on maximum performance.

But you can make a **self-written** simple "switch" to simplify this code:
```c++
    Color color = UnknownColor;
    if (argv[i] == "red") {
        color = Red;
    } else if (argv[i] == "orange") {
        color = Orange;
    } else if (argv[i] == "yellow") {
        color = Yellow;
    } else if (argv[i] == "green") {
        color = Green;
    } else if (argv[i] == "violet" || argv[i] == "purple") {
        color = Violet;
    }
```
into this:
```c++
    Color color = StringSwitch<Color>(argv[i])
        .Case("red", Red)
        .Case("orange", Orange)
        .Case("yellow", Yellow)
        .Case("green", Green)
        .Cases("violet", "purple", Violet)
        .Default(UnknownColor);
```

There is a `StringSwitch` realization in llvm: [StringSwitch.h](https://github.com/llvm/llvm-project/blob/main/llvm/include/llvm/ADT/StringSwitch.h).

This class has just [two fields](https://github.com/llvm/llvm-project/blob/506e93687140a1be054a825ae08f512d3ebbdbf8/llvm/include/llvm/ADT/StringSwitch.h#L45-L50 ):
1. `std::string_view str` - the string (`argv[i]` in our case)
2. `std::optional<T> value` - the resulting value (in our case `T == Color`)

When calling the [Case method](https://github.com/llvm/llvm-project/blob/506e93687140a1be054a825ae08f512d3ebbdbf8/llvm/include/llvm/ADT/StringSwitch.h#L68-L74),
if `value` is not set yet and the given string equals to `str`, then `value` is set.

There are [EndsWith and StartsWith methods](https://github.com/llvm/llvm-project/blob/506e93687140a1be054a825ae08f512d3ebbdbf8/llvm/include/llvm/ADT/StringSwitch.h#L76-L88)
which will fill `value` if a part of `str` equals to the given string.

There are [corresponding case-insensitive methods](https://github.com/llvm/llvm-project/blob/506e93687140a1be054a825ae08f512d3ebbdbf8/llvm/include/llvm/ADT/StringSwitch.h#L141-L161),
as well as [Case methods for multiple values](https://github.com/llvm/llvm-project/blob/506e93687140a1be054a825ae08f512d3ebbdbf8/llvm/include/llvm/ADT/StringSwitch.h#L90-L139).

Finally there is a [conversion operator](https://github.com/llvm/llvm-project/blob/506e93687140a1be054a825ae08f512d3ebbdbf8/llvm/include/llvm/ADT/StringSwitch.h#L188-L191)
(in our case it converts to `Color`).

In my opinion, one can also make a `LambdaSwitch` class, which, unlike `StringSwitch`, should accept lambdas, and set a value if the given lambda returns true. 🙂

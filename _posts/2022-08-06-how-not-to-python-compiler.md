---
title: (Python) How not to write a compiler 🌾🚜🐄
date: 2022-08-06 17:18:00
tags: [compiler]
categories: [Article]
---

There is a brand new Python compiler, which is being discussed on Reddit.

[https://www.reddit.com/r/Python/comments/w7vlim/i_made_a_python_compiler_that_can_compile_python/](https://www.reddit.com/r/Python/comments/w7vlim/i_made_a_python_compiler_that_can_compile_python/)

[https://github.com/Omyyyy/pycom/tree/main](https://github.com/Omyyyy/pycom/tree/main)

It doesn't make sense to compile (more precisely, to translate) Python into C++, because the languages are radically different.

*With best efforts* one can translate the bare minimum of language's features.
For example, this code `val = 1; val = 'hello'` (where types are dynamically changed) won't translate to C++.
But let's look into details, what the "compiler" does.

It's upsetting that the repository has **more that 1000 stars** and a lot of positive comments on Reddit, despite being a nonviable tool.

It uses the built-in Python lexer ([the tokenizer library](https://docs.python.org/3/library/tokenize.html)
to get tokens, and then it would iterate over the tokens and just translate one word in Python to another word in C++:
[compiler.py](https://github.com/Omyyyy/pycom/blob/6242343e1afb1259ec7f81def7abace1eb7f55c2/src/pycom/compiler.py#L277-L330).
This makes it unable to "compile" almost any program different from "Hello World", the translator's logic is not flexible.

This task is better done via the built-in Python lexer+parser ([the ast library](https://docs.python.org/3/library/ast.html)).
The AST uses the "visitor" pattern: the library "visits" every node of the tree and generates the code.
Almost all tools that work over source code (including translators) use the visitor pattern.
The translator could look like this - [gist.github.com](https://gist.github.com/Izaron/677af3ad27f9effa017f0ca87e20c3c0).

The next questionable choise is that C++ is needed as the "intermediate representation" just to make sure that we get an optimized executable file.
This is also not so fine idea - it's better to translate Python into [LLVM IR](https://releases.llvm.org/6.0.0/docs/tutorial/LangImpl03.html),
because it is more flexible than C++ (from compiler perspective).
You can even find some projects on this topic, when searching `python llvm`.

In general, translating Python into another representation to "optimize" it doesn't make sense - many of Python libraries
are actually written on C/C++ and just provide a lightweight Python API!

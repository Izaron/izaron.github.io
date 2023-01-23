---
title: Your own compiler - do it yourself ⚙️🛠
date: 2022-06-12 22:01:00
tags: [compiler, story]
---

In my opinion, there is the best tutorial to make your own programming language compiler (in C++):
[My First Language Frontend](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html).

The theory and practice of writing compilers is a whole world, which has been evolving for a few decades.
Enthusiasts implement their own programming languages (here is [an interesting collection](https://proglangdesign.net/)),
often alone.

The tutorial tells how to write a compiler for a simple Turing-complete language, which is capable to execute
programs like this:
```
# Compute the x'th fibonacci number.
def fib(x)
  if x < 3 then
    1
  else
    fib(x-1)+fib(x-2)

# This expression will compute the 40th number.
fib(40)
```

The tutorial explains compilers' pipeline and these topics:
1. [Lexical analysis](https://en.wikipedia.org/wiki/Lexical_analysis) - translating the file into tokens.
2. [Parsing](https://en.wikipedia.org/wiki/Parsing) - translating the tokens into an AST (Abstract Syntax Tree)
with writing a simple [LL(1)-parser](https://en.wikipedia.org/wiki/LL_parser).
3. Code generation - translating the AST into the [LLVM IR](https://llvm.org/docs/LangRef.html) (intermediate representation);
you'll learn theory about [SSA](https://en.wikipedia.org/wiki/Static_single-assignment_form)
and [control-flow graph](https://en.wikipedia.org/wiki/Control-flow_graph).
4. Code optimization - tweaking optimization passes (there is [the list of the passes](https://llvm.org/docs/Passes.html)) and
theory about it.
5. Compilating the code into an object file.
6. ... and many other topics: debug-symbols, types, memory handling, standard library...

The C++ is ultimately hard in each of these topics (and has non-standard phases like the preprocessor stage 🤯), but
with using a simpler language one can learn how compilers work and even how to create your own compiler.

I liked the most the possibility to link your language's code with C++ code: I made a simple description
[on github](https://github.com/Izaron/kaleidoscope#linking).

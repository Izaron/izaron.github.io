---
title: Is it possible to call main() recursively?
date: 2022-06-13 17:37:00
tags: [creepy]
---

(Some useless info, I guess)

Long time ago I was reading "questions for C++ interview" and met this question: "it is possible to call the `main()` function
from the program, that is, recursively?"

```c++
int main() {
    int n;
    std::cin >> n;
    if (n != 0) {
        return main();
    }
    return 0;
}
```

Well, ~~what's the purpose of doing this~~ why one couldn't do this?

In all environments, which I'm aware of, the `main()` function is not the actual [entry point](https://en.wikipedia.org/wiki/Entry_point) -
firstly they call a compiler-generated function that initializes the global variables. The `main()` doesn't
differ from other function from this point of view.

The `main()` function linkage is *implementation-define* ([link to the Standard](http://eel.is/c++draft/basic#start.main-3)): the name
of the function doesn't [get mangled](https://github.com/llvm/llvm-project/blob/b9a7dea9171416a998e4fa3333fb9f76baa167b8/clang/lib/AST/ItaniumMangle.cpp#L705-L707).
Also the Standard mentions that one shouldn't call the `main()` function inside the program.

The Standard clarifies that recursivelly calling `main()` is prohibited ([link to the Standard](http://eel.is/c++draft/expr#call-13)).

But every modern compiler compiles such code successfully! They could emit a warning if it's compiled with the `-Wmain` flag:
[link to godbolt](https://godbolt.org/z/dWc3bnTs1).

Therefore the answer to the question is as follows: it is prohibited to call `main()` recursively, but the compiler will compile such code
without complaints, if you don't use flags like `-Wmain` or `-pedantic`.

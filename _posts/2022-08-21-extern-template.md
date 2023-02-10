---
title: Don't compile template code on every build ⌨️
date: 2022-08-21 18:39:00
tags: [compiler, advice]
categories: [Article]
---

I have a "childhood trauma", I don't really like templates -
I think it's better to use them in exceptional cases.
There are not many cases where classes/methods *must* be templated.

I use my own indicator: if the template parameters are theoretically unknown in advance, then it is needed (for example,
if this is a container type like `std::vector<T>`).

If all parameters are known (for example `void make_sound<TAnimalCat/TAnimalDog/TAnimalBird>()`), then it is better to make a virtual class `IAnimal`.

But we often have to work with an established architecture, so let's imagine that we have templates for a finite number of parameters.

The template code itself doesn't do anything yet.
Only when a method/class with some template parameters is called, the template is "instantiated", that is,
a new method/class is generated for these parameters.

When compiling each `.cpp` file (a project may have hundreds of them),
we often have to compile **the same piece of code**: [example on godbolt](https://godbolt.org/z/7565KfaE4).

Instantiated template functions have the [linkonce_odr](https://releases.llvm.org/8.0.1/docs/LangRef.html#linkage-types) linkage type,
you may want read [my article about linkage types](/posts/practical-linkage-types/).

To avoid compiling the same code in each `.cpp` file,
you can "*declare* instantiations" for all known parameters using `extern template`: [example on godbolt](https://godbolt.org/z/fq1bKEz4v).

In this case, somewhere else you need to "*define* instantiations" - for the example above, you need to write this in some `.cpp` file:
```c++
    template int calc<float>();
    template int calc<double>();
```
Now the code in the template will be compiled only once (one time for each instantiation).

However, you can also keep all the template code in your `.cpp` file, this often simplifies readability: [example on godbolt](https://godbolt.org/z/7hbhqn6bf).

You can evaluate the usefulness of different approaches:
- The `extern template` approach is not a full solution, because usually the gain in compilation speed is absolutely insignificant.
- The approach with the template code completely in its `.cpp` file is not bad, it improves the readability of the code.

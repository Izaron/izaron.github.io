---
title: Practical linkage types 🔗
date: 2022-06-05 17:41:00
tags: [compiler]
categories: [Articles]
---

The standard way to define a functions is as follows: one writes the *declaration* in an `.h`-file (so that
many translation units get aware of its existence) and the *definition* in a `.cpp`-file.

To fully define the function (declaration+definition) in an `.h`-file there is the `inline` keyword used,
but one can use the `static` keyword as well.

Let's see the LLVM IR (the intermediate representation of the C++-code) for `int sum1()`, `static int sum2()`,
`inline int sum3()`: [link to godbolt](https://godbolt.org/z/c9nPjTMr1).

The LLVM IR specifies the interesting list of symbols' [linkage types](https://llvm.org/docs/LangRef.html#linkage-types),
there are many types. The linkage type defines how the symbol behaves when the linker is working.
(A symbol is either a function or a global variable)

- `sum1` has the default linkage type, `external` - there is exactly 1 definition of the symbol is possible.
The linker fails if the program contains 0 or >1 definitions.
- `sum2` has the `internal` linkage type - the symbol is reachable only in the translation unit where it was defined.
The linker just renames the internal symbol if there is names collision.
- `sum3` has the `linkonce_odr` linkage type. This is a typical [weak symbol](https://en.wikipedia.org/wiki/Weak_symbol),
similar to some other types (`linkonce`, `weak`).
The program can have multiple weak definitions of the same symbol. If all the definitions are weak, the linker takes
a random weak definition. If one of the definitions is strong (like `sum1`), the linker takes the strong definition.

What is the difference between `linkonce` and `linkonce_odr`?
- Weak definitions might differ from each other, therefore, for example, the compiler has no right to inline a call of a weak
function (the function may point to another definition after the linker stage).
- But the C++ Standard requires that the programmer secures that inline functions have exactly one definition
(this is usually true because the definition is placed in an `.h`-file).
- Therefore the compiler **has** the right to inline a call to `sum3` - nothing could break because of this.

Let's compiler the code into an object file:
```shell
clang++ -c link.cpp
```
We'll get an [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) file on Linux. Let's use the `readelf` tool
to observer the symbol table. To print human-readable function names
(not [mangled ones](https://en.wikipedia.org/wiki/Name_mangling#How_different_compilers_mangle_the_same_functions))
the `c++filt` tool can be used:
```shell
readelf -s link.o | c++filt
```
We get kind of this (I omitted other symbols):
```
Symbol table '.symtab' contains 21 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     6: 00000000000000c0    18 FUNC    LOCAL  DEFAULT    2 sum2(int, int)
    14: 0000000000000000    18 FUNC    GLOBAL DEFAULT    2 sum1(int, int)
    20: 0000000000000000    18 FUNC    WEAK   DEFAULT    7 sum3(int, int)
```

In general, it is rather better to use inline functions than static functions. This way the executable won't contain
many copies of the same function. Also, this way the executable will have exactly one instance of static variables inside
the function.

```c++
inline int* get_address() {
    // `dummy` takes sizeof(int) memory
    // it could be N*sizeof(int) if it was static function
    static int dummy;
    // inline-function: returns the same pointer everywhere
    // static-function: returns a unique pointer for every translation unit
    return &dummy;
}
```

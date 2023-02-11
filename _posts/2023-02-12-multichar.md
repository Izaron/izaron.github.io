---
title: char, but not char 🤔
date: 2023-02-12 11:00:00
tags: [creepy]
categories: [Article]
---

When you explore a file format, you may find *file signatures*. A file signature is
a sequence of bytes that used to verify the content of a file. A file signature may look as a sequence of "magic bytes" (unique for this type) at the
beginning of a file.

There is a list of file signatures for some formats [on wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures).
Bytes at the beginning of the file are often human-readable, meaning that they encode Lating letters - for example
`Rar!`, `LZIP`, `OggS`.

Let's look at an example class that takes 4 bytes and then can check whether it's the file signature of a RAR-compressed file:
```c++
class BinarySignature {
public:
    BinarySignature(int32_t value)
        : Value_{value}
    {}

    int32_t AsInt() {
        return Value_;
    }

    bool IsRar() {
        return Value_ == 'Rar!';
    }

private:
    int32_t Value_;
};
```

Do you see something creepy? Yes, this is comparing an `int` with a multi-symbol `char`!
```
Value_ == 'Rar!'
```

The `'X'` literal is supported everywhere and has the `char` type.

The `'XXXXX'` literal has the `int` type, but the compiler has the right to not support literals of these types.
Also this literal has the *implementation-defined* numerical value, that is, it's up to the compiler to decide
what `int` value generate from this literal.

Most of C++ compilers support the multi-symbol char literal and convert it to the `int` value as it were consecutive
bytes of the corresponding int value.

[Link to godbolt](https://godbolt.org/z/MeGYadzqr).

So you now know about this feature of C++ and its cool use-case when you work with human-readable signatures
in binary files. 🙂

---
title: Dormitory pointers
date: 2022-02-10 14:43:00
tags: [madskillz]
categories: [Articles]
---

Let's say we want to have a function that returns an object with some boolean flags in addition.
These flags serve as the object's characterictics. Then we should have something like this:
```c++
template<typename Ty> class ActionResult {
    bool Invalid = false;
    Ty T;
    // some methods...
};
```

But this doesn't fit some high-end projects (such as Clang) because of some memory overhead (due to struct's alignment).
If we do a lot of work with pointers, and the said type `Ty` is often a **pointer type**, then we can tailor `ActionResult` exclusively
to pointer types.

In this case we can transfer the boolean flags... Right inside the pointer:
```c++
template<typename PtrTy> class ActionResult {
    uintptr_t PtrWithInvalid; // let's assume sizeof(uintptr_t) == sizeof(PtrTy*)
};
```

How does it work? A pointer type is 8 bytes long on 64bit machines, and is capable to address up to 16mlb terabytes of RAM.
Here comes that some bits of these 8 bytes (the significant ones) are not used at all, they're knowingly equal to zero.

We can transfer the `Invalid` flag into the 0th bit:
```c++
     bool isInvalid() const { return PtrWithInvalid & 0x01; }
     bool isUsable() const { return PtrWithInvalid > 0x01; }
     bool isUnset() const { return PtrWithInvalid == 0; }
```
and shift the pointer to one bit, making room for the flag:
```c++
     PtrTy get() const {
       return reinterpret_cast<PtrTy *>((PtrWithInvalid & ~0x01) >> 1);
     }
```

Clang utilizes pointers with up to three boolean flags (three bits): [class PointerIntPair](https://github.com/llvm/llvm-project/blob/main/llvm/include/llvm/ADT/PointerIntPair.h).

One of the most useful example is the `QualType` class. It contains the pointer to a type (`Type*`) and
qualificator flags (`const`, `restrict`, `volatile`) along with the pointer.
Due to the technic described above, these flags don't utilize more memory at all!

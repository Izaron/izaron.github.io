---
title: Branch tables for switch operators
date: 2022-12-28 11:02:00
tags: [compiler]
math: true
categories: [Article]
---

In C++, the `switch` operator is used to transfer the control flow to different locations depending on the value of the given variable.
The `switch` statement is a correspondence between the values of the variable, and the code that must be executed depending on the value.

Depending on the target architecture,
optimization level, properties of the given switch operator,
the code can be generated in two different forms. The forms are:
1. A chain of consecutive `if`s. This is the easiest way, because the switch operator is always representable in this form.
2. A *branch table* (aka *jump table*).

The first option is not interesting, it is the simplest and most unoptimized.
If there are $30$ `case`s in our `switch`, then in the worst case there will be $30$ (!) consecutive comparisons
(a chain of `if`s) before the program understands the location of the code to jump to.

<details>
  <summary>If the compiler is smart enough</summary>
  In fact, in such cases, compilers are able to do *à la* "binary search", so there will probably be $log_2(30)$ comparisons in the worst case. 😁
</details>

In the second option, the address of the instruction to jump to is
calculated immediately depending on the value of the variable, and no comparison will be performed.

An example of `switch` with branch table: [Link to godbolt](https://godbolt.org/z/3debYb4vq).
In this example, the `switch` works on the enum value.
For the compiler, enum is represented as its [underlying type](https://en.cppreference.com/w/cpp/types/underlying_type).
By default, this type is `int`, that is, in all operations with enum, the implicit conversion to `int` occurs.

Thus, we can imagine that this is a `switch` on values from $0$ to $6$ inclusive.

In the example, the compiler generated the labels `LBB0_2`, `LBB0_3`, ..., `LBB0_8` for each corresponding `case X`.

The compiler also made the table `LJTI0_0`, which holds the addresses of these labels. Well, "table" is a big word, it's just our abstraction.
The "table" just consists of several consecutive 8-byte numbers, which are the addresses of the said labels `LBB0_2`-`LBB0_8`.
And the `LJTI0_0` label indicates the beginning of the sequence.

Now, having the "address table", the compiler can calculate the address where the runtime needs to jump.
If the parameter is 0, then it jumps to the first address of the table, if 1 then to the second, and so on.
```
        lea     rcx, [rip + .LJTI0_0]
        movsxd  rax, dword ptr [rcx + 4*rax]
        add     rax, rcx
        jmp     rax
```

*Off-topic*: Labels make sense only for assembler.
The label is just a conditional address in the executable (pointing to an instruction or data).
After the linker made its work (when separate object files are packed into one executable file), normal hardcoded addresses appear instead of labels.
```
        lea     rcx, [rip + 0x012345678]
```

The address table may have a slightly different implementation, but written above is the general idea.
For example, in a [Wikipedia example](https://en.wikipedia.org/wiki/Branch_table#Example)
in a random 8-bit assembler, the value of the variable is added to the command address register
(`addwf PCL,F`), and immediately after this instruction there is a table with goto-s to the desired instruction, and the command address register will point to the desired goto
instruction.

The compiler determines itself whether the address table is needed.
It is usually used for "dense" `switch`es,
where there is a `case X` for consecutive values of `X`.
If you put random values in `case X`, then the tables will not work - [link to godbolt](https://godbolt.org/z/hGvGT61Tv), there will be consecutive `if`s.

---
title: A myth about virtual destructors
date: 2022-11-21 20:31:00
tags: [creepy]
categories: [Articles]
---

There is a question that occurs often at interviews and in real life: "Why do we need the virtual destructor?"

In a very ~~reliable~~ source of knowledge (that is, on the Internet), almost everywhere is written in this way:
> If you have at least one virtual method in your class, the destructor should also be made virtual.
At the same time, do not forget that the destructor will not be virtual by default,
so you should declare it explicitly. If this is not done, you will almost certainly have UB in your program (most likely in the form of memory leaks).

But there is a logical question: why then wouldn't the compiler generate a virtual destructor **automatically** (for classes with virtual methods)? 🤔
After all, he generates a bunch of things (for example, move- and copy-constructors, move- and copy-assignment operators).

The answer is: because the statement above is incorrect! 😱
The virtual destructor is needed only when there is a risk that an object of a derived class can be destroyed using a pointer to the base class.

Suppose there is a base virtual class `DrinkMachine` and its derived class `CoffeeMachine`.
Each object of the program must be destructed sooner or later, this is done via 2 different paths:
1. The object was constructed on the stack, then the program will call the destructor automatically:
```c++
{
    CoffeeMachine cm;
    // they will call cm.~CoffeeMachine() on the scope exit
}
```
In this case, it does not matter whether the destructor is virtual or not,
because in both cases the destructor of the real object will be called.
2. The object is constructed on the heap, then the programmer assures the destruction of the object, usually like this:
```c++
DrinkMachine* dm = ...; // perhaps there was `new CoffeeMachine()` in place of `...`
// ...
delete dm;
```
(If we have an object like `std::unique_ptr<DrinkMachine>`, then the same thing happens via RAII)

The `delete` expression usually does two things: calling the destructor and freeing memory:
```c++
dm->~DrinkMachine();
std::free(dm);
```

In this case, **it is important** that the destructor of the real object is called,
that is, perhaps we would like to call `~CoffeeMachine()`. In this case, **the virtual destructor is needed** -
the real destructor will be found at run-time in the [vtable](https://en.wikipedia.org/wiki/Virtual_method_table).

If the second case show above does not occur in the program, then the virtual destructor is not needed - for example, everything works without errors in this program:
```c++
void MakeDrink(DrinkMachine& dm) {
    dm.MakeDrink(); // calling the virtual method
}
void MakeCoffee() {
    CoffeeMachine cm;
    MakeDrink(cm);
    // cm deletes automatically by calling `cm.~CoffeeMachine()`
}
```

This may be important for programs that need to be optimized,
because calling a virtual method (including the virtual destructor) triggers an overhead in the form of two memory loads.

P. S. There is such a thing as **devirtualization** -
when the compiler understands in advance which method just to call
(without looking at the vtable in run-time).
But to do this, the compiler must prove that it knows exactly which method to call.
[There is a good article on this topic](https://blog.feabhas.com/2022/11/using-final-in-c-to-improve-performance/).

Devirtualization **is not standardized**, so you need to check on your compiler yourself -
whether calling the virtual destructor in the example above (with `MakeCoffee`) is optimized or not.
If yes, then you may do not worry and always make the virtual destructor.

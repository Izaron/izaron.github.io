---
title: The most vexing C++ rule for modular projects (and how to deal with it)
date: 2023-01-06 11:54:00
tags: [creepy, compiler]
---

While developing my pet project,
I encountered what I call the most vexing C++ rule in my experience,
at least for modular programs.
It is linker-specific and requires sort of secret knowledge to deal with it.

In large modular projects, often each business logic module corresponds to a single static library.

If some module is not linked to the executable binary file (via a flag of the build system or whatever),
then this module will not be built, and nothing will break in the executable.
It's just that there will be fewer features in the executable.
On the other hand, if a module is linked, then it must somehow "register" itself in the list of modules. 😐

Say we have the `module.h` file (simplified):
```c++
    struct IModule { virtual void Do() = 0; };
    void AddModule(std::unique_ptr<IModule> module);
    const std::vector<std::unique_ptr<IModule>>& GetModules();
```

The `main.cpp` file ought to use the `GetModules()` function, and a module ought to use the `AddModule()` function.

The only way this can be done is to add the code that should be called at the start of the program.
This is done through static initialization of objects in the object's constructor of the helper objects.
So, somewhere in one of the `.cpp` files of the module there should be such code:
```c++
    struct Dummy {
        Dummy() {
            AddModule(std::make_shared<MyCoolModule>());
        }
    };
    static Dummy dummy;
```

And then the shitshow begins. The `dummy` variable (and its type's code) get into the `libcoolmodule.a` static library
(you can observer it via [objdump](https://en.wikipedia.org/wiki/Objdump)), but **the linker throws the variable out
as it thinks it is unused**.
Eventually, the module is not going to be registered.

The fun thing is that the process of solving this problem depends on the build system,
operating system and many other things.
There is no general solution of the problem.
How this problem can be solved in different environments:
1. Windows: mark variables that linker is forbidden to ~optimize out~ throw out:
```c++
__pragma comment(linker,"/include:?variable_name@")
```
2. Place the variables into a special section and mark the section as sort of non-optimizable:
```c++
static Var_t g_DumbVar __attribute__((__used__, section(".var_section.g_DumbVar"))) = (const Var_t) X_MARKER;
static Var_t* g_DumbVarGuard[] __attribute__((__used__, section(".guard"))) = { &g_DumbVar };
```
3. Linux: mark a variable as non-optimizable via some linker command parameters.
3. Linux: write a link script for the linker which configures how to link things.

After a while of searching the solution, I found the answer to why the linker works this way.
When you build a binary, the individual object files (the `.o` files) which passed to the linker are linked in order and nothing in them is thrown away.

But there is an "optimization" on libraries (the `.a` files, by the way every library is just an archive of `.o` files).
A `.o` file from the library is linked only if it contains the definition of some undefined symbol that is required by previously linked `.o` files.
Otherwise, it is assumed that this `.o` is not needed and it does not link into executable.

So, the CMake build system has a hack to bypass this rule. One can replace this line:
```
add_library(enum_serializer STATIC module.cpp helper.cpp)
```
with this one:
```
add_library(enum_serializer OBJECT module.cpp helper.cpp)
```
And then, if some executable target depends on the `enum_serializer` module, the linker will link not the `libenum_serializer.a` file,
but rather the `module.o` and `helper.o` files.
Therefore, the "registration" of the module will work and the problem is solved 😁

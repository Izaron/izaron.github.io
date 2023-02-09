---
title: How does the compiler measure compilation speed? 🕰
date: 2022-08-20 10:25:00
tags: [compiler]
categories: [Articles]
---

In C++, it's very easy to make that a project takes very long time to build.
In my personal top, a unit test (in the form of a single `.cpp` file) used to compile for 4.5 minutes.

Fortunately, the compilation speed is debuggable.
When compiling a single file, you need to specify the
[-ftime-trace](https://releases.llvm.org/12.0.0/tools/clang/docs/ClangCommandLineReference.html#cmdoption-clang-ftime-trace) setting:
```
-ftime-trace
    Turn on time profiler. Generates JSON file based on output filename. Results can be analyzed with chrome://tracing or Speedscope App for flamegraph visualization.
-ftime-trace-granularity=<arg>
    Minimum time granularity (in microseconds) traced by time profiler
```

The command could look like this:
```shell
clang++ main.cpp -c -ftime-trace -ftime-trace-granularity=50
```

The resulting `main.json` file can be visualized on 🔬[Speed Scope](https://www.speedscope.app/).
(There is a GIF on its [github](https://github.com/jlfwong/speedscope#speedscope) presenting an example of visualisation).

What does the compiler do:
- ⚙️ Before starting compilation, if the `-ftime-trace` setting is set, clang will call the
[llvm::timeTraceProfilerInitialize](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/clang/tools/driver/cc1_main.cpp#L216-L221) method.
- ⚙️ In this method, an object of the
[llvm::TimeTraceProfiler](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/llvm/lib/Support/TimeProfiler.cpp#L292-L293)
is initialized.
- ⚙️ When an event starts, you need to call the
[llvm::TimeTraceProfiler::begin](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/llvm/lib/Support/TimeProfiler.cpp#L105-L108)
method to remember the starting time.
- ⚙️ When the event ends, you need to call the
[llvm::TimeTraceProfiler::end](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/llvm/lib/Support/TimeProfiler.cpp#L110)
method to complete an event record.
- ⚙️ As you can see from the code, they use a stack because the events are nested inside
each other (for example, there can be a "parse class" event inside a "file compilation" event).
- ⚙️ After compiling the file, the
[llvm::TimeTraceProfiler::write](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/llvm/lib/Support/TimeProfiler.cpp#L148)
method is called to create a json file.

By default, the `-ftime-trace-granularity` parameter equals to `500` (500 microseconds).
Not all events are recorded, but only sufficiently "long" ones that lasted longer than 500µs -
[the code that checks it](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/llvm/lib/Support/TimeProfiler.cpp#L125-L127).

In the code, the necessary methods are not called "manually" - the standard RAII idiom is used in the form of the
[llvm::TimeTraceScope](https://github.com/llvm/llvm-project/blob/c56846a8928f8708f56c0eb36dcd6345e312faa0/llvm/include/llvm/Support/TimeProfiler.h#L130-L134)
structure.
As you can see, at the moment of calling the constructor "the event begins", calling the destructor "the event ends".
(if compilation was invoked without the `-ftime-trace` flag, then this object does nothing)

There is an example: this is how the time for template instantiation is measured (which occurs after parsing the file):
[PerformPendingInstantiations](https://github.com/llvm/llvm-project/blob/804d4594cbe217ae817b6786b0e9965283f78aa2/clang/lib/Sema/Sema.cpp#L1081-L1084).

While templates are being instantiated, all sorts of "nested" events are recorded, for example,
[InstantiateFunction](https://github.com/llvm/llvm-project/blob/804d4594cbe217ae817b6786b0e9965283f78aa2/clang/lib/Sema/SemaTemplateInstantiateDecl.cpp#L4880).

That's how the compiler supports the flame graphs in a simple way 🙂

In my experience of observing the compilation speed, the "frontend" of the compiler
(which parses a file in AST) takes **3-20 times** longer than the "backend" (which translates the AST to LLVM IR, makes optimizations, and translaties to binary).
The main reason for this imbalance is the huge size of the source file after all `#include`s are done (this is true for almost all modern C++ projects).

Based on this data, it becomes clear what needs to be corrected to speed up compilation. However, speeding up the compilation is a completely different story...

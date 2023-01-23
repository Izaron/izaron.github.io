---
title: Garbage Collector
date: 2022-02-12 13:45:00
tags: [madskillz]
---

There are C++ projects with garbage collection.

For example, the **Blink** browser engine (part of Chrome). It's a monolithic project, where dependencies between
different objects are so hard that it is unable for a human to keep the whole picture in mind.
And there are A LOT of mental cyclic dependencies. To avoid physical cyclic dependencies, one had to write code
like this:
```c++
class A {
    RefPtr<B> m_b;
};
class B {
    A* m_a;
};
```

That's it. At some point it was so bad that memory was leaking in *10% of tests*. The problem was solved by introducing
a garbage collector. It haven't affected the performance much, but eliminated most of memory leaks and crashes - it's a win!

[Link to the Google Doc with the GC presentation](https://docs.google.com/presentation/d/1YtfurcyKFS0hxPOnC3U6JJroM8aRP49Yf0QWznZ9jrk/edit).

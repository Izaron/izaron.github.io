---
title: Iterators with conditional end 🏁
date: 2022-08-15 11:01:00
tags: [madskillz]
categories: [Articles]
---

Iterators are one of the core concepts of C++.
C++ has its own corresponding iterator class for each container class (set/vector/list/...) used to access its own data.

The class for which the corresponding iterator class is defined ought to have the `begin()` and `end()` methods, which return the iterator objects.
The iterator class ought to have the `opeator++()` and `operator*()` methods.
The presence of these methods is sufficient to use the iterator in various standard methods and in the
[range-based for loop](https://en.cppreference.com/w/cpp/language/range-for).

Usually iterators iterate over all objects from `begin()` to `end()`:
```c++
    const std::vector<int> vec{1, 3, 5, 7, 9};
    // the code below is equivalent to `for (int value : vec) { /* ... */ }`
    auto __begin = vec.begin();
    auto __end = vec.end();
    for ( ; __begin != __end; ++__begin) {
        int value = *__begin;
        /* do something with `value`... */
    }
```

However, there are cases when the `end()` *cannot be calculated in advance*
and it is necessary to check at every step whether we should exit the loop.

In the C++20 standard, there is such an approach:
1. Introduce an empty class [std::default_sentinel_t](https://en.cppreference.com/w/cpp/iterator/default_sentinel_t).
2. The `end()` method of the container class should return an object of the mentioned empty class:
```c++
    std::default_sentinel_t end() { 
        return {}; 
    }
```
(the `begin()` method will keep returning a normal iterator object)
3. The iterator class must define a comparison operator with objects of the empty class:
```c++
    bool operator==(std::default_sentinel_t) const { 
        return /* some condition */; 
    }
```

As the result, the old code with iterators works as before, but ends only when the comparison operator returns `true`!

What are the real use cases:
- 🎯 [std::counted_iterator](https://en.cppreference.com/w/cpp/iterator/counted_iterator) - a wrapper above
another iterator object, which iterates over no more than `N` objects.
- 🎯 ["Range-based for" loop for coroutines](https://godbolt.org/z/G5cWP57b8) -
to be able to take new values from the picture in a loop while it is active (the iterator class is `class Iter`).

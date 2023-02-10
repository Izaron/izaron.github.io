---
title: Heterogeneous Lookup
date: 2022-05-11 14:26:00
tags: [creepy]
categories: [Article]
---

What happens when you try to call `.find()`/`.contains()`/etc on an associative container
(`std::set`/`std::map`/their unordered versions) using a key of a different type? 🤔

```c++
std::map<std::string, int> m;
// ...
if (m.contains("foobar")) { ... }
```

The [std::map](https://en.cppreference.com/w/cpp/container/map) has the default comparator as the third template
parameter - `std::less<Key>`. The comparator provides an operator which is called while searching in the container.

The comparator is a struct with `bool operator()`. It looks like this:
```c++
constexpr bool operator()(const Key &lhs, const Key &rhs) const {
    return lhs < rhs;
}
```

That is, the argument's type must be the same as the key type. Looking back at the beginning:
the `"foobar"` arguments has type `const char[7]`. It can decay into the `const char*` type or casted to
`std::string_view` or casted to `std::string`. Therefore, unfortunately, after overload resolution it is needed to create
a temporary object of type `std::string`. 💩

We can use a string type with a custom allocator to check whether the temporary objects are created
(look at [std::string definition](https://en.cppreference.com/w/cpp/string)). The custom allocator will log allocation requests.
```c++
using traceable_string = std::basic_string<char, std::char_traits<char>, trace_allocator<char>>;
```
The strings should be long enough to get around Small String Optimization and make an actual memory allocation. In my
environment it should be at least 16 chars long: [godbolt link](https://godbolt.org/z/3s5rPv6sd).

So, the problem is that we don't want to create `std::string` temporary objects, because different string types are comparable to each other.
The problem was solved in C++14 via an **angry hack** 🪓: they made the default template parameter `std::less<> = std::less<void>` and
specialized the `std::less<void>` template:
```c++
template <class Key1, class Key2>
constexpr auto operator()(Key1&& lhs, Key2&& rhs) const
    return static_cast<Key1&&>(lhs) < static_cast<Key2&&>(rhs);
}
```

Now everyone has to remember about `std::less<>` and declare containers like this:
```c++
std::map<std::string, int, std::less<>> m;
```
then there won't be temporary objects with memory allocations: [godbolt link](https://godbolt.org/z/KMYKhdzhb).
This hack is what implied by fancy words *heterogeneous lookup*.

I couldn't find the reason why containers don't use `std::less<>` by default. I even checked the STL-like libraries with container
realizations and saw two appoaches:
1. Homogeneous lookup by default (like in STL) - example [boost::container::map](https://www.boost.org/doc/libs/master/doc/html/boost/container/map.html).
2. Heterogeneous lookup by default - example [hash table in Abseil](https://abseil.io/docs/cpp/guides/container) (from Google),
it's an std::unordered\_map/set replacement.

The performance is better with heterogeneous lookup, there are [benchmarks on unordered containters](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0919r3.html#implementation) - it gives speedup between 20% to 35%.

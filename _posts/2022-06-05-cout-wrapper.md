---
title: Wrapper for output streams 🌊
date: 2022-06-05 19:52:00
tags: [madskillz]
categories: [Article]
---

Every big tech company (like FAANG) has its own implementation of `std::string` and/or
a string utils library.

All over in the code, if you want to send a slightly changed string to the output stream
(an output stream is an object with the `<<` operator, like `std::cout`, `std::stringstream`),
there is a new string created:
```c++
log << "Value is " << valueStr.Quote();
```
The `Quote` method here creates a new string with quotes `"` on the left and the right end of the string.
We meet this code everywhere and it harms performance⏱, but this looks better than printing ugly `"\""`.

Let's try to create a "flag" for performance optimal string printing, just like [std::boolalpha](https://en.cppreference.com/w/cpp/io/manip/boolalpha).
We want to write like this:
```c++
log << "Value is " << quote << valueStr;
```
so it is the equivalent of this:
```c++
log << "Value is " << '"' << valueStr << '"';
```

How to do this? Let's say this expression: `((stream object) << quote)` returns a `TQuoteWrapper` objects, and this expression:
`((a TQuoteWrapper object) << str)` returns the original stream object with `str` written there.

We can work with any stream object. The `quote` object means nothing and serves only to make possible the expressions above.
```c++
inline constexpr struct quote_t{} quote;

template<typename TStream>
auto operator<<(TStream& stream, quote_t) {
    return TQuoteWrapper{stream};
}
```

The `TQuoteWrapper` class has the `<<` operator:
```c++
    template<typename TArg>
    TStream& operator<<(TArg&& arg) {
        return Stream_ << '"' << std::forward<TArg>(arg) << '"';
    }
```

[Link to godbolt](https://godbolt.org/z/6vvdq7K9r) with this class and examples.

Also we can implement that the `TQuoteWrapper` escapes string chars (for example, replacing `\"` to `\\\"`).
[Link to godbolt](https://godbolt.org/z/7GsrdraMh) to that implementation.

Another approach to performance-friendly string streaming is in the **abseil** library - [Writing to Stream](https://abseil.io/docs/cpp/guides/format#writing-to-a-stream).
There we pass a "light" object to the stream:
```c++
std::cout << absl::StreamFormat("\"%s\"", valueStr);
```
But there is a constraint - you can't atritrarily customize the string (like escaping its chars).

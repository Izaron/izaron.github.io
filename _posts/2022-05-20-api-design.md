---
title: 📚 Book review&#58; "API Design for C++" (2011)
date: 2022-05-20 11:00:00
tags: [books]
---

![](/assets/img/posts/2022-05-20/cover.jpg){: .w-25 .left}

[Link to Amazon](https://www.amazon.com/API-Design-C-Martin-Reddy/dp/0123850037).

The teamlead of a neighboring team recommended me this book. He absolutely loved it 🤩 But I had controversial impressions 🤔

What is an API? The book says:
> An API is a logical interface to a software component that hides the internal details required to implement it.


We are surrounded by different APIs, even a single program has multiple APIs. The book explores questions related to API design in C++.

The book quite clearly separates "general principles of API design" and "specific issues of API design in C++".

The first component, as the philosophical one, is indeed very nice -- there are a lot of sensible ideas, hints, as well as examples of the API of popular projects.
The book helps you understand what issues a software architect is thinking about so that the API does not become non-usable in a few weeks 👍

The second component, as the technical one, in my opinion, is weak. Here are some of the cons that I didn't like:
- The book was published in 2011, it describes features of `C++0x` (the working title of `C++11`) of an ancient version.
The `auto` keyword is nowhere used in the book, they write `std::vector<double>::iterator` by hand, and so on.
- Some paragraphs are written by Captain Oblivious, for example `Avoid #define for constants`.
- Very little said about ABI compatibility, exactly one page with trivial information. Without mentioning tools like
[abidiff](https://www.sourceware.org/libabigail/manual/abidiff.html) and specific recommendations.
- Static and dynamic libraries are described only in the last 10 pages of the 450-pages long book, also with trivial information.

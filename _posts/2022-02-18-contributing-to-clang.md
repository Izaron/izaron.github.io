---
title: Contributing to Clang - impressions and thoughts
date: 2022-02-18 13:26:00
tags: [compiler]
categories: [Article]
---

Last week I tried to fix some bugs in Clang, just to see how it would work out. I had found
some [issues](https://github.com/llvm/llvm-project/issues?q=is%3Aissue+is%3Aopen+consteval) related
to `consteval` functions and had been sending micro fixes in review requests.

![](/assets/img/posts/2022-02-18/clang.jpg)

I've made four patches total (every not bigger than a handful of lines), you can see them on the screenshot
(not counting the clang-tidy patch).

One of them had been approved within a week, the others are waiting on review ~~and will be waiting a long time~~.
My impressions and thoughts:

1. Opened GitHub issues might keep being opened for many years before someone takes and fixes it.
This period probably depends on the issue's importance, but there seemingly no KPI on reducing
issues in Clang/LLVM.
2. A rule from the official guide -- when you're making a patch, find the reviewers yourself.
You can inspect git blame, similar issues and so on, in order to find relevant people. Sometimes the patch is reviewed by some random reviewers,
but it might be not the case usually -- you *are* to find active reviewers.
3. The reviewers (5-6 of them who were reviewing my patches) are **VERY** seasoned in C++. After searching their names on Google,
you'll see them among authors of C++ Standard's proposals, CppCon speakers, C++ commitee members, and so on.
4. The contributors' community is not that big, there are a handful of very active contributors who review most of patches. If you're
up to fix a specific part (for example related to `consteval`), it can be ~2 contributors who made most of the consteval code, and
anybody else won't have enough expertise to review your patch.

In general, thinghs don't go fast. But this is a volunteer movements, and we should respect people who make the world
better on their unpaid time.

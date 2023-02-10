---
title: 📚 Book review&#58; "Modern C++ Programming with Test-Driven Development" (2013)
date: 2022-11-28 12:14:00
tags: [testing, books]
categories: [Article]
---

![](/assets/img/posts/2022-11-28/cover.jpg){: .w-25 .left}

[Link to The Pragmatic Bookshelf](https://pragprog.com/titles/lotdd/modern-c-programming-with-test-driven-development/).

### Introduction

In the last few years, I have elaborated my own rule: the set of tests describes all possible use-cases that the program should be able to accomplish.

This means that if there is no test that describes some behavior, then this behavior does not exist or it is not documented (or the tests are incomplete).

It sounds super trivial,
but this approach reduces the number of surprises per square meter and
most likely will not allow you to take part in such stupidity as fixing the same bugs several times.

It also means that if you have written some code,
then you ought to assume that the new code is incorrect until there are tests that prove the opposite.
A normal situation is when several times more time is spent on writing tests for the new code than on the new code itself,
because the loss of time and resources due to a bug in the prod is still many times higher than the time spent to writing tests.

This was my introduction! And the book that I will review here, describes the application of the
[test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) in C++.
The TDD means work in short cycles, in each cycle you firstly write a test for new behavior, then you write the implementation hat satisfies the test just written.

Despite the name, the TDD is more about the design of the system, because it forces you to make the program interface as testable as possible.

In general, from the book you can understand what TDD is. They use [GoogleTest](https://github.com/google/googletest) for examples.
Unfortunately, there are disadvantages in the book:

### Excessive redundancy

The same thing is repeated like ten times, as if the book is being articifially inflated. There is also a lot of text written by the Captain Obvious.

For example, in the section *"Running the Wrong Tests"*, the question is -
what to do, if you ran tests, but the new test did not start?
The answer: you may have run the wrong test suite, or you have the wrong filter, or you have not compiled the tests, or the test is turned off.

In the section *"Testing the Wrong Code"* - you may be testing wrong code if you forgot to compile the module, or compilation failed and you didn't notice it.

After spending 10 minutes on some trivialties, you're gonna reading the book diagonally 😤

### Overcomplicating

In a chapter of 50 pages long, they analyze an example of a trivial module,
which was made with using to TDD.
Instead of the first approximate solution (as people do it in the real world),
the author makes wacky edits to pass the new next case and no more - it ends up with rewritting everything a hundred times.

The author gives awkward advices *à la* "Martin Fowler":
> Your favorite tests contain one, two, or three lines with one assertion.
Tests with no more than three lines and a single assertion take only minutes to write and usually only minutes to implement

Which frankly has nothing to do with reality.

### Poor usage of technologies

Very little written about code coverage and CI.
The execution time of the methods is measured by printing the current time to the terminal,
although there is the cool [Google Benchmark](https://github.com/google/benchmark) library for this purpose
(for benchmarking small code snippets) and [Valgrind](https://en.wikipedia.org/wiki/Valgrind) for the entire program.
The [Dependency Inversion](https://levelup.gitconnected.com/dependency-inversion-principle-in-c-14ec84408201)
(so that you can use mocked classes in tests) is called a *complex concept*.

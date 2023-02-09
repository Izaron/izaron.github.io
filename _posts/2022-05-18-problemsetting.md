---
title: How to prepare a programming contest
date: 2022-05-18 19:08:00
tags: [story]
categories: [Articles]
---

![](/assets/img/posts/2022-05-18/acm.jpg)

Probably a lot of coders ever tried to solve tasks from programming contests: in
[codeforces.com](https://codeforces.com/?locale=en), school olympiads, ACM ICPC, or somewhere else.
Most people use C++ to solve tasks.

I used to enjoy solving these tasks and even occasionally [prepared tasks for codeforces](https://codeforces.com/contests/writer/Sehnsucht?locale=en)
and for school olympiads. It is called "problemsetting". I have prepared around 15-20 tasks.

This work is paid quite symbolically.
Therefore, this is mainly done by students -- former top contest solvers who have not yet graduated from
university and have not found a decent job yet.
Then they get bored and are replaced by a new generation of students.

I'll show example of problemsetting on this task: [Codeforces Round #704 - E. Almost Fault-Tolerant Database](https://codeforces.com/contest/1492/problem/E?locale=en).
This is the hardest task of the round despite its simple description.

People prepare tasks mainly on [polygon.codeforces.com](https://polygon.codeforces.com/). This is the most advanced problemsetting
service, and has been the only working service for a long time. The other problemsetting services (CodeChef, Yandex.Contest, etc.)
have borrowed features from it -- like validation, embedded version control system, and so on.

To write small programs (generators, checkers) [testlib.h](https://codeforces.com/blog/entry/18289?locale=en) is used.

The task preparation process consists of these steps:
1. Invention of the unique problem idea in a short formulation.
2. A fancy description of the task is created in LaTeX - [example](https://gist.github.com/Izaron/f8910eaee53260bdf32a2814dcd94c0c), this is called the **legend** of the task.
3. Test cases **generators** are written, this can be a handful of simple scripts --
[example 1](https://gist.github.com/Izaron/47e710f292770486622d1ea2dab856b6),
[example 2](https://gist.github.com/Izaron/699c89d5449f65291fe96ba85fe8ac0c).
They are run with different arguments to generate test cases.
4. The **validator** is written. It is needed to check that the test cases have the right format
and satisfy the constraints described in the legend - [example](https://gist.github.com/Izaron/a5c5f5b2faff9b59c0d00400f81430e9).
5. Test cases are generated using the [Freemarker](https://freemarker.apache.org/) template engine.
It is possible to write a script that transforms to generator calls - [example from docs](https://gist.github.com/Izaron/a1e7015eb38a042a522e2132438f23c0).
Our task has 100 tests - [the list](https://gist.github.com/Izaron/74954bf543d65ecffab3bb5fb855857f).
6. The author's **solution** is written. There is the only one reference solution - [example](https://gist.github.com/Izaron/b0f2084357a594d846155d9b07de8c0b),
but some deliberately wrong solution (with wrong idea or slow) are also to be written.
They are needed to make sure that no wrong solution passes all the tests.
7. The **checker** is written. It is needed to check the participants' solutions correctness.
The polygon service has some default checkers for simple output formats, but usually it is needed to write
a unique checker. It is especially needed when the task could have more than one correct answer -
[example](https://gist.github.com/Izaron/3c28fb7a74668cd918efe6b52d84efb6).

This is a quite serious job, so fellow problemsetters should conduct code review of your scripts and solutions
as well as proofreading of the legend.

About 5-15 people of different problem solving skills solve the new task without knowing the solution in advance.
They can come up with an unexpected simpler solution, or pass all the tests with a wrong solution -
all these cases are considered separately.

According to the law of large numbers, epic fails of this kind do occur for one out of dozens of tasks:
1. A large fraction of participants came up with a simpler solution, so the complexity of the task is actually smaller.
2. The task has weak tests, so many "wrong" or "slow" solutions pass all the tests.
3. The author's solution is incorrect.

The first two kinds of epic fails are somewhat acceptable, but the third one is a complete shitshow, and
Codeforces cancels the ongoing contest immediately if this is detected (more precisely, the contest
is declared as "unrated", and it makes little sense to participate in it at all).

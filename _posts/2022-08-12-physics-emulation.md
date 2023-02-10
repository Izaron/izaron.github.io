---
title: Simulating physics in C++ using numerical analysis 🌊
date: 2022-08-12 11:02:00
tags: [story]
math: true
categories: [Article]
---

Programs for all kinds of simulations (water, fire, smoke, glass, heat exchange, etc.)
take a large part in the world of C++. In addition to academic research, this is applied in films and games.
While studying at the university, I used to do something similar during classes (using the university's supercomputer and NVIDIA GPUs).

First, the physical process must be described by a mathematical model. This is a theory that is taught step by step in universities:
- 📚 General theory of [differential equations](https://en.wikipedia.org/wiki/Differential_equation) and everything related to it (usually a 1-year course at a university).
- 📚 [Vector calculus](https://en.wikipedia.org/wiki/Vector_calculus) which, however,
was not a separate course in my case, but was studied as part of mathematical analysis towards the end of the 1.5-year course.
- 📚 From physics, people usually take some well-known formulas - it's enough to study materials in the needed field for several months (hydrodynamics/electromagnetism/...)
to understand the theory completely.
- 📚 The course of [mathematical physics](https://en.wikipedia.org/wiki/Mathematical_physics) combines previous steps and explores some equations in the 0.5-year course.

What do some of the simplest equations look like (without "sources of heat" and other influences):
- 🔬 [Heat equation](https://en.wikipedia.org/wiki/Heat_equation) (there is a GIF with simulation);
- 🔬 [Wave equation](https://en.wikipedia.org/wiki/Wave_equation) (simulation [video on YouTube](https://www.youtube.com/watch?v=wlT_PHF1zng)).

What is generally needed for computer simulation of a physical situation, along with a mathematical model?

Modeling takes place on a finite domain (in terms of space and in time), therefore,
we need correctly specified initial and/or boundary conditions - usually the model describes the state at time $t = 0$ and,
possible, the state at the boundaries of the domain at each moment $t$.

This is enough for simulation - that is, the computer does not need to look for an analytical solution to a mathematical model.
In some cases, it is even impossible to get an analytical solution - the notorious example is
the [Navier-Stokes equations](https://en.wikipedia.org/wiki/Navier–Stokes_existence_and_smoothness) (for simulating liquids or smoke).

The area of the simulated phenomenon is represented as a **grid**.
Imagine that in a two-dimensional model we operate on an area of $NxM$ centimeters.
Then we need to choose the "step" $h$ cm, so that we get an array of $\frac{N}{h} \times \frac{M}{h}$ points.

Now all the functions of the model are *discretized*, that is, we calculated their values only at these points.

Now derivatives for these functions can be calculated on using the neighboring points.
Here is what can be substituted instead of $f'(x)$ (the derivative of $f(x)$):

$\frac{f(x+h) - f(x - h)}{2h}$

And this is what you can substitute instead of a second-order derivative $f''(x)$:

$\frac{f(x+h) - 2f(x) + f(x - h)}{4h^2}$

Using this simple method, you can get formulas to calculate the entire evolution of a physical state 🔥

This is called a [difference scheme](https://encyclopediaofmath.org/wiki/Difference_scheme).

The science that explores difference schemes, their inaccuration, and ways to solve them quickly is called
[numerical analysis](https://en.wikipedia.org/wiki/Numerical_analysis). There is a bunch of methods for optimizing these solutions.

Many simulations are well parallelizable,
they can be reduced to a matrix form,
and can be successfully calculated on supercomputers or on GPUs - this is another skill that needs to be mastered
if you work in this area📚

You can use a library like [SFML](https://www.sfml-dev.org/) for beautiful graphics.
However, if three-dimensional graphics is planned, then you need to learn the same amount of knowledge in the field of Computer Graphics.

This is the way to learn how to simulate physical processes, with proper understanding what is happening inside 🙂

A bonus - simulating aerodynamics of different geometries 🎥:

{% include embed/youtube.html id='bJX8fVsq5oQ' %}

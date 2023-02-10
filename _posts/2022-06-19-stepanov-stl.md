---
title: Story of Alexander Stepanov, the author of STL 🔆
date: 2022-06-19 18:25:00
tags: [story, retro]
categories: [Article]
---

An example: for modern children, smartphones have always existed, push-button phones are something ancient,
and wired phones are just an attribute from films about the cold war.

Likewise, for programmers who started their journey 10-15 years ago, the
[STL](https://en.wikipedia.org/wiki/Standard_Template_Library) has always existed.
However, STL has an interesting history behind it.

I came across the Wikipedia page about [Alexander Stepanov - the author of the STL](https://en.wikipedia.org/wiki/Alexander_Stepanov).
I read it, as well as a lot of interviews with him, and some of his books.
There is a collection of his materials on [stepanovpapers.com](http://stepanovpapers.com/).

In my opinion, Alexander's biography begins in an unusual way:
- Born in 1950 in Moscow
- In 1972 he graduated from the Moscow State University (degree in maths)
- From 1972 to 1976 he worked as a programmer (developing a mini-computer for controlling large hydroelectric power stations)
- In 1977 he emigrated to the USA

I have little idea how many such qualified programmers there were in the world in 1972 (!)
who actually were able to write their own debuggers, linkers, real-time operating systems. They must had been unique specialists at this time.

I also wonder how it was possible to emigrate from the USSR to the USA in 1977 with such an important background,
given the stories about unusual escapes in those years
([In Russian, but the online translator will do fine](https://pikabu.ru/story/istorii_samyikh_gromkikh_i_neobyichnyikh_pobegov_sovetskikh_grazhdan_iz_sssr_5630952).
Unfortunately, this question is not touched upon in any interview🙁 Only some banal phrases like in a 2003 interview:
> I left the USSR because I didn't like the Soviet government, and Microsoft is the Soviet government in programming.

Anyway, let's return to the interviews and books. Any interview will be interesting because it reveals some little-known facts.
Alexander is generally the creator of [generic programming](https://en.wikipedia.org/wiki/Generic_programming)
as a conception. Before C++, he implemented this approach in the Scheme and Ada languages.

You can read [this interview](http://www.stlport.org/resources/StepanovUSA.html) - there are jokes, for example,
a question "does STL stand for "**St**epanov and **L**ee"?"; as well as the fact that first ideas about generic programming came to
Alexander in 1976 in a semi-conscious state when he was lying in the hospital with food poisoning.
Also Alexander in this interview expresses his opinion about the uselessness of OOP and Java (defined as *money-oriented programming (MOP)*).

Of the books, I liked his [Notes fir the Programming course at Adobe 2005-2006](http://stepanovpapers.com/notes.pdf) the most.
At the beginning Alexander describes how he taught himself the principles of good code in 1972-1976.
First he was writing a project (for example, a debugger), and after a few months he has to rewrite it from scratch with knowledge of
all design errors, then it was repeated, and so on.
Then he wrote code not randomly, but taking into account some principles
(the length of functions no more than 20 lines, refactoring common code into common functions, etc.)
This experiment was extremely successful and greatly simplified development.

Alexander noticed that the larger the program, it has the less *application-specific* (or *non-general*) code and more *general* code.
In modern desktop applications, the fraction of *non-general* code is much less than 1%.
Nowadays this is self-evident if you look at the amount of code of the operating system and Qt/WinForms.
But at that time it was not so obvious, because the development was very close to the hardware.

Nowadays generic programming is commonplace, but in those days it was quite a countercultural idea.
It does not correspond to the concept of "top-down design", because it is the idea that it is possible to design
a good component without much knowledge about how this component will be used 🤔

From the book mentioned above, you can read the first few chapters - there is very rare information on how to design generic classes correctly
(like `std::vector`), and about how different C++ places should look from Alexander's point of view (but Stroustrup did not allow).
Then there in less interesting part begins: reinvention of `std::move` (the book was written in 2006), and it doesn't make sense to read it further.

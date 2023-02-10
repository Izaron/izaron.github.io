---
title: Thin client for C++ developers (with screenshots!)
date: 2022-10-04 21:26:00
tags: [story]
categories: [Article]
---

I haven't posted anything in the blog for a while, but now I have moved, got a WiFi, so I decided to write a post related to thin clients 😎

Year by year the size of big projects grows up, and it becomes impossible or very hard to develop them on a personal computer.

Let's look at Clang/LLVM: as seasoned contributors remember, ~5 years ago it was possible to build it in Debug mode on a standard PC. Nowadays it's not possible, because it eats so much RAM while linking binaries so even 16GB may be not enough.

Therefore people build in in Release or RelWithDebInfo mode - but one can't debug the code in these modes, has to plant a lot of debug logs (like `cerr() << Expr->size()`), so it's hard to do something non-trivial 🤷

One may have a powerful PC, but this is inconvenient (if you like to work while lying in bed) and one can't move the big PC quickly somewhere else.

I personally haven't been developing something serious on the PC for many years. I use virtual servers at work, and my private virtual server for my pet projects.

There is a number of providers from which you can obtain a private virtual server - Google Cloud, Microsoft Azure, AWS, Yandex Cloud, etc., depending on benefits you want and money you have. For example, I have a server on Yandex Cloud, with these resources:

![](/assets/img/posts/2022-10-04/resources.png)
*🔍 Virtual server resources (sorry, cyrillic letters)*

It has an `Intel Cascade Lake` processor, 32 cores, 32GB RAM, 200GB SSD. It costs me ~11540 roubles (~168$) per month (providers have a calculator, so you can tweak the parameters to know the cost in advance).

These resources are enough for everything - for example, Clang/LLVM builds in Debug mode from scratch in only 13 minutes (several times faster in comparison with a PC) and you're able to actually debug in via gdb/lldb.

You can choose any OS of any version for your virtual machine, and it is also hundred times faster than installing them on a PC

![](/assets/img/posts/2022-10-04/list.png)
*🔍 The list of OSes*

You can enter the virtual server from a local PC via SSH

![](/assets/img/posts/2022-10-04/ssh.png)
*🔍 Part of .ssh/config and entering the virtual server*

In my case I run `ssh -A mango` in the terminal.

Of course, if you haven't worked a lot of time with virtual servers, it's firstly hard to work effectively with it. I'll describe which tool I have used for many years.

I use [byobu](https://www.byobu.org/) as the terminal window manager.

![](/assets/img/posts/2022-10-04/byobu.png)
*🔍 byobu windows manager inside the virtual server*

I use [Neovim](https://neovim.io/) (fork of Vim) to write the code, but it's a matter of habit. Most of my colleagues use [Visual Studio Code](https://code.visualstudio.com/), it can connect to virtual servers via SSH, and is just more familiar to most developers.

![](/assets/img/posts/2022-10-04/neovim.png)
*🔍 Neovim (I have contextual hints and autocomplete)*

I use LLDB (a GDB analogue) to debug the code

![](/assets/img/posts/2022-10-04/lldb.png)
*🔍 lldb working*

I use version control systems also inside the virtual machine

![](/assets/img/posts/2022-10-04/git.png)
*🔍 git inside the virtual machine*

So, the developer doesn't have to get out of the powerful virtual server, and his notebook (as the thin client) may be not so powerful but rather comfortably. Whenever I'm going to buy a new notebook, I choose a light notebook so it will fit in a backpack nicely.

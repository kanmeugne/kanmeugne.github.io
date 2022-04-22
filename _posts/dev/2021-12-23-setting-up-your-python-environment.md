---
sidebar: right
title:  "Setting Up Your Python Environment"
teaser: "How to set up your python environment quickly"
categories:
    - software development
tags:
    - python
    - jupyter
    - ipython
    - programming
comments: true
author: kanmeugne
---

I have been talking python with differents kind of programmers -- from beginners to experienced programmers -- and there is this little issue which, surprisingly, does not have a straightforward answer : **what do you need to install to start programming in python ?** This post is dedicated to this litlle issue and I will try explain what -- in my opinion -- is the easiest path from... nothing... to a decent python programming environment. 

![Introduction au TDD Poster](/images/test11-unsplash.jpg){: width="500"}
_Photo by [Todd Quackenbush](https://unsplash.com/@toddquackenbush?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)._


Our journey will have several *stages*  -- and you will have to do some installs homework to do in order to move from one stage to another.

**At the first stage**, you will have to make sure that you are working on the right Operating System and with the right terminal -- meaning, compatible with this post. Spoiler Alert! We will go for a linux OS since it is very popular among programmers and easy to set up.

**At the second stage**, you should have installed `python 3` on your computer -- we will see that, once installed, you technically have a working python programming environment. We will show off two programming modes at this stage : and *interactive mode* -- where you can type commands and see the result in an interactive prompt -- and *script mode* -- where you run scripts using the python interpreter.

**At the third stage**, we will enrich our programming environment with *high-level* python prompts -- like `ipython` and `jupyter` -- and advanced IDEs -- like `vscode`. High-level prompts give more context to programmers and are clearly a most-have if you want to improve your interactive programming mode experience. IDEs, as you might expect, improve the script programming mode experience with embedded tools for : script execution, coding style, code highlighting, testing, etc.

Now, let's begin...

## 1. The OS : where everything starts

![Linux OS](/images/gabriel-heinzer-4Mw7nkQDByk-unsplash.jpg){: width="500"}
_Photo by [Gabriel Heinzer](https://unsplash.com/@6heinz3r?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/linux-terminal?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_

I will assume that you are working on a Linux machine. The first reason I am confident with this assumption is that Linux is a very popular OS for developers (beginner and experienced ones) and Microsoft (which is the most popular OS among normal human being) provides a descent linux subsystem which is an Ubuntu distribution by default. I am thus pretty sure that if the reader (this is you) is a windows user (which is highly possible), he will be able to set up an Linux machine, even on his windows computer, with little effort -- the interested reader could check this [article][1] to understand how to set it up.

The second reason I am confident is that, if you are a Mac user (which is highly possible), most of the setup and commands I will show in this post will work on your OS with very little or no change at all. Anyway, for the sake of straightforwardness, I will assume a linux os -- ay be I will focus on MacOs in another post.

## 2. The Python interpreter : the mandatory install

![Python Language](/images/alex-chumak-zGuBURGGmdY-unsplash.jpg){: width="500"}
_Photo by [Alex Chumak](https://unsplash.com/@ralexnder) on [Unsplash](https://unsplash.com/s/photos/python-code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_

Python is actually an interpreter, which means that, unlike a compiler, a python code is parsed and executed dynamically by a special program which is : the **Python Interpreter**.

If you have a linux OS setup, it is highly possible that you already have a `python` version installed on your system. But you need to install the *right* version and make sure to be able to track it down -- as you might expect, the interpreter parsing and executing behaviour depends on its version. So how do you install the right python interpreter ? Since you are working on a full linux system (as we have assumed above), you can do the following steps :

1. Check if a `python3.X.X` is already installed. You can type the following command on your terminal :
   
   ``` bash
   $ python --version
   ```
   If the result of this command looks like `python X.Y.Z` where `X<3` or if the command fails, you should probably install `python 3.X.X`. It is highly recommended since most of the python modules and tools provided by the community will not be maintained for older versions. So how do you install `python 3` ?

2. You need to install `python 3` and the exact *how* depends on your linux distribution. For Ubuntu distribution (WSL runs Ubuntu by default), the following instructions should be sufficient (for `python 3.8`, but you can pick any `3.X` version you want).
   
   ```bash
    $ sudo apt-get update
    $ sudo apt-get install python3.8
   ```
   Anyway, I am not going to re-invent the wheel, so I encourage you to [read this post][3] and get the right instructions for your linux distribution. Come back here when you are done with `python 3` installation.


Now, you should have the right version installed. Technically, you are ready to start programming in Python -- you are ready to experience two different programming modes : the **interactive mode** and the **script mode**.

### Interactive Mode

### Script Mode


[1]: https://www.geek4geeks.com
[2]: https://www.python.org "see the available version"
[3]: https://docs.python-guide.org/starting/install3/linux/
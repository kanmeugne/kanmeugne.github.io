---
sidebar: right
title:  "Setting up your python environment"
teaser: "How to set up your python environment quickly"
categories:
    - software development
tags:
    - python
    - jupyter
    - ipython
    - programming
comments: true
image: /images/test11-unsplash.jpg
author: kanmeugne
---

I have been talking python with differents kind of programmers -- from beginners to experienced programmers -- and there is this little issue which, surprisingly, does not have a straightforward answer : *what do you need to install to start programming in python ?* This post is dedicated to this litlle issue and I will try explain what -- in my opinion -- is the easiest path from nothing to a decent python programming environment. Our journey will have several *stops* and you will have to install *stuff* in order to move from one stop to another. 

At the first stop you will have to make sure that you are working on the right Operating System, with the right terminal -- meaning, compatible with this post. Spoil Alert! We will go for a linux OS since it is very popular among programmers.

At the second stop, you should have installed `python 3` on your computer -- we will see that, once installed, you technically have a working python programming environment. We will show off two programming mode : and interactive mode -- where you can type commands and see the result in an interactive prompt -- and scripted mode -- where you write scripts and run them with the python interpreter.

At the third stop we will enrich our environment with high-level python prompt -- like `ipython` and `jupyter` -- and advanced IDEs -- like `vscode`. High-level prompts give more context to the programmers and are clearly a most have if you want to improve your interactive programming mode experience. IDEs, as you might expect, improve the script programming mode experience with embedded tools for : script execution, coding style, code highlighting, testing, etc.

## 1. The OS : where everything starts

I will assume that you are working on a Linux machine. The first reason I am confident with this assumption is that Linux is a very popular OS for developers (beginner and experienced ones) and Microsoft (which is the most popular OS among normal human being) provides a descent linux subsystem -- read this [article][1] to understand how to set it up -- which is an Ubuntu distribution by default. I am thus pretty sure that if the reader (this is you) is a windows user (which is highly possible), he will be able to set up an Linux machine, even on his windows computer, with little effort.
The second reason I am confident is that, if you are a Mac user (which is highly possible), most of the setup and commands I will show in this post will work on your OS with very little or no change at all. This said, for the sake of understandability, I will fully cover MacOs installation on another post. 

So get your Linux OS ready and let's go !

## 2. The Python interpreter : the mandatory install

Python is actually an interpreter, which means that, unlike a compiler, a python code is parsed and executed dynamically by a special program which is : the Python Interpreter. You need to install the right version of this interpreter and make sure to be able to track its version down -- python syntax (and utilities) is highly dependent on the version of the python interpreter you are using to process it.

So how do you install the right python interpreter ? (As we have assumed in the previous section, I will consider that you are working from either a WSL (linux subsystem for windows) or a linux (any distribution) terminal :

1. Check if a `python3.X.X` is already installed. You can type the following on your terminal :

``` bash
$ python --version
```

If the result of this command looks like `python X.Y.Z` where `X<3` or if the command fails, you should probably install `python 3.X.X`. It is highly recommended since most of the python modules and tools provided by the community will not be maintained for older versions. So how do you install `python 3` ? Jump to the next point to see the answer.

2. This is where problems begin -- you have to install `python 3` but the *how* depends on your linux distribution. Since I don't want to re-invent the wheel, I encourage you to [read this post][3] and get the right instructions for your linux distribution. Come back here when you are done (WSL runs Ubuntu by default).


Now, you should have the right version installed so technically you are ready to start programming in Python. It is still possible to improve the programming environment though. Let's write some code first before going further.


[1]: https://www.geek4geeks.com
[2]: https://www.python.org "see the available version"
[3]: https://docs.python-guide.org/starting/install3/linux/
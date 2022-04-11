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

I have been talking python with differents kind of programmers -- from beginners to experienced programmers -- and there is this little issue which surprisingly, does not have a straightforward answer : *what do you need to install to start programming in python ?* I decided that this should be the main concern of this post as I want to spend some time explaining the easiest path from nothing to a decent python programming environment. So let's start the journey.

## 1. The OS : where everything starts

I will assume that you are working on an Ubuntu machine. The first reason I am confident with this assumption is that Ubuntu is a very popular OS for developers (beginner and experienced ones) and Microsoft (which is the most popular OS among normal human being) provides a descent linux subsystem -- read this [article][1] to understand how to set it up -- that runs an Ubuntu by default. So I am pretty sure that if the reader (this is you) is a windows user (which is highly possible), he will be able to set up an Ubuntu machine with little effort. The second reason I am confident with this assumption is that almost everything about python in this post will work for any other linux distribution. So for linux user, don't worry if you want to stick to the best distribution (we all know that yours is the best) -- you should be able to follow. The third reason is that if you are a Mac user (which is highly possible), most of the setup and commands I will show in this post will work on your OS with very little or no change at all.

So get your os ready and let's move to the next step.

## 2. The Python interpreter : the mandatory install

Python is actually an interpreter, which means that, unlike a compiler, a python code is parsed and executed by a specific program which is : the Python interpreter. You need to install the right version of this interpreter and make sure to be able to track it down -- python syntax (and utilities) is tightly coupled with the version of the python interpreter you are using to process it.

So how do you install the python interpreter ? I recommend you to go through through the following steps : 
- first of all, check if a `python3.X.X` is already installed. You can type the following on a terminal if you are an linux user :

```bash
    $ sudo apt update
    $ sudo apt install python3.X.X  # replace 3.X.X by the version you need
```
If you are a mac user, 

On an Ubuntu machine, you can type the following command in order to install the desired version of python. 

This could sound very simple, but actually it is not. Most of ubuntu system (and even macOs system) are shiped with a pre-installed python interpreter
[1]: https://www.geek4geeks.com
[2]: (https://www.python.org "see the available version")
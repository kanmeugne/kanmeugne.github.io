---
layout: page
sidebar: right
title:  "Modeling pheromon evaporation on a 2D Grid with SFML"
teaser: "In a previous article, I presented a C++ SFML application that models a 2D Grid with the possibility to add and remove obstacles. In this post, I am going to add a very nice feature : <i>pheromon evaporation</i>. I will discuss how this kind of feature could profit operational research and online optimization modelers."
tags:
    - sfml
    - cmake
    - c++
    - modeling
    - pheromon
    - evaporation
categories:
    - modeling & simulation
image:
    thumb: SFMLCMAKE-thumb.png
---

## Introduction

You should now be familiar with our [2D Grid app][1] so I won't introduce it again. In a [previous post][1], I updated the architecture in order to implement the add/remove obstacles feature. Here, I am going to upgrade the architecture again in order to implement another feature : pheromon.

You can see pheromon like a chemical substance that is dropped somewhere, as a mark of a given organic activity, and then evaporates overtime. This the basic way to implements a stigmergic behavior --- which is a type of behavior based on indirect communication through a common and shared space. Stigmergy explains the emergence of collective behavior in social insects colonies, where one can witness very complex organization among intellectually limited agents. The concept has been first introduced by the french biologist [Pierre-Paul Grassé][2] and systematically studied by [Deneubourg][3] for different ants species. In the following section I am going to detail the architecture that allows me to mimic an ant-like pheromon evaporation. Basically the objects are the same, comparing to the previous architecture. I am just going to add another viewer for pheromon -- `PheromonViewer` -- and more controls in the `App` object -- `App::addPheromon`. `IGrid`, and consequently `Grid`, will be upgraded in order to pheromon related methods. Finally, I will introduce the `App::evaporate` function, which will be responsible of the evaporation process.

## Managing pheromon in the 2D Grid

In his article, [Craig Scott][3] explained how to build *GoogleTest* and *GoogleMock* directly in a *CMake project*. The configuration can be used almost as-is to add *SFML* dependencies as an *external project*.

The main idea is to invoke *CMake* `ExternalProject` *command* and perform the build at *configure time*. This fully integrates the *external project* to your build and gives you access to all the *targets*. The author uses two sets of rules to configure the *build*.

## Configure and build the project

Now that we have our project files ready, we need to type the following lines in a terminal at the root of the project folder (**with the right generator**).

**Note**: If you are on linux, you might need to check [this][2] first.

{% highlight shell %}
  # on windows
  $ cmake  -G "Visual Studio 15 2017" -S . -B ./build 
  $ cmake  --build ./build --config Debug --target app

  # on linux
  $ mkdir build  
  $ cd build
  $ cmake -G "Unix Makefiles" .. -DCMAKE_BUILD_TYPE=Debug
  $ cmake --build ./ --target app 
{% endhighlight %}

At the end of the stage, you should ba able to launch the executable located in the bin folder.

{% highlight shell %}
  bin> ./Debug/app
{% endhighlight %}

![screenshot](/images/sfmlcmake.jpg)

Now you are ready to take your graphical app everywhere you want. Enjoy and feel free to send me your feedbacks!

## More like this
{: .t60 }
{% include list-posts tag='sfml cmake c++' %}

[1]: https://github.com/kanmeugne/sfmlcmake
[2]: https://www.sfml-dev.org/tutorials/2.5/compile-with-cmake.php
[3]: https://crascit.com/2015/07/25/cmake-gtest/
[4]: https://github.com/google/googletest
[5]: https://cmake.org/
[6]: https://www.sfml-dev.org/documentation/2.5.1/
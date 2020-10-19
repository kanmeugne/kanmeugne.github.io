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

You should now be familiar with our [2D Grid app][1] so I won't present it again. In a [previous post][1], I updated the architecture in order to implement the add/remove obstacles feature. Here, I am going to upgrade the architecture again in order to implement pheromon management.

You can see pheromon like a chemical substance that is dropped somewhere, as a mark of a given organic activity, and then evaporates overtime. It is a very productive way to implement a *stigmergic* behavior --- which is a type of behavior based on indirect communication through a common and shared space. Stigmergy explains the emergence of collective behavior among several social species with limited intellectual capacities. The concept has been first introduced by the french biologist [Pierre-Paul Grassé][2] and systematically studied by [Deneubourg][3] for different ants species.

In this post, I am going to detail the architecture that allows me to mimic an ant-like pheromon evaporation. The objects are the same, comparing to the [previous architecture][1]. Simply put, I am going to do the following : 

1. add another viewer for pheromon --- *PheromonViewer* --- and more controls in the *App* object -- *App::addPheromon*
2. upgrade *IGrid*, and consequently *Grid*,  to declare and implement pheromon related methods
3. define a new method --- *App::evaporate* --- responsible of the evaporation process.



## More like this
{: .t60 }
{% include list-posts tag='sfml cmake c++' %}

[1]: https://github.com/kanmeugne/sfmlcmake
[2]: https://www.sfml-dev.org/tutorials/2.5/compile-with-cmake.php
[3]: https://crascit.com/2015/07/25/cmake-gtest/
[4]: https://github.com/google/googletest
[5]: https://cmake.org/
[6]: https://www.sfml-dev.org/documentation/2.5.1/
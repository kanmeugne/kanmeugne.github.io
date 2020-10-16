---
layout: page-fullwidth
title:  "Pheromon evaporation on a 2D Grid"
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

You should now be familiar with our [2D Grid app][1] so I won't present it again. In a [previous post][1], I updated the architecture in order to implement the add/remove obstacles feature. Here, I am going to upgrade the architecture again in order to implement a pheromon evaporation feature.

You can see pheromon like a chemical substance that is dropped somewhere --- as a mark of an organic activity --- and then evaporates overtime. It is a very productive way to implement a *stigmergic* behavior --- which is a type of behavior based on indirect communication through a common and shared space. Stigmergy explains the emergence of collective behavior among several social species with limited intellectual abilities. The concept has been first introduced by the french biologist [Pierre-Paul Grassé][2] and systematically studied by [Deneubourg][3] for different ants species.

The proposed improvements of the previous [object-oriented architecture][1] --- will focus on mimicking and illustrating an ant-like pheromon evaporation. Simply put, I am going to do the following : 

1. add another viewer for pheromon --- `PheromonViewer` --- and more controls in the `App` object -- `App::addPheromon`
2. upgrade `IGrid` --- and consequently `Grid` --- to declare and implement pheromon related methods
3. define a new method --- `App::evaporate` --- responsible of the evaporation process

{% plantuml %}
@startuml
title
<size:10>Fig. 1. Upgraded Architecture with pheromon manipulation and visualization features.</size>
<size:10>There is on more object (PheromonViewer) and several others have been updated (<i>IGrid, IGrid::CELL, Grid, App)</i></size>
end title
package geometry <<Frame>>
{
}
interface ICellFunctor 
abstract class AbstractViewer 
class App 
class CELL  
class ViewerMgr
class ObstacleViewer extends AbstractViewer
class PheromonViewer extends AbstractViewer
class GridViewer extends AbstractViewer
interface IGrid 
class Grid implements IGrid 
IGrid ..> CELL : defines >
Grid *--> CELL 
App o-> IGrid : controls >
App o-- AbstractViewer
AbstractViewer <---o ViewerMgr
ViewerMgr --|> AbstractViewer
IGrid ..> ICellFunctor : defines >
ICellFunctor ..> CELL
PheromonViewer ..> ICellFunctor : runs >
ObstacleViewer ..> ICellFunctor : runs >
GridViewer ..> geometry
hide members
{% endplantuml %}

## Pheromon modeling and evaporation

The `App` object is augmented with `App::addPheromon` and `App::evaporate` methods both responsible of *adding* a little amount of pheromon in a selected cell, and *evaporating* pheromons over time --- see Fig. 2. `App::evaporate` will take a *time interval* as parameter in order to schedule the *evaporation process* (we will use [SFML *clocks*][4] to implement this --- see the [code][2] for more details). A special method to apply evaporation is also defined in `IGrid` --- `IGrid::iUpdatePheromon`.

{% plantuml %}
@startuml
title: <size:10>Fig. 2. App and IGrid improvements</size>
class App {
    + {static} const int DEFAULT_HEIGHT
    + {static} const int DEFAULT_WIDTH
    + {static} const int DEFAULT_RESX
    + {static} const int DEFAULT_RESY
    + void run()
    + void display()
    + bool addObstacle(int, int)
    + bool removeObstacle(int, int)
    + bool addPheromon(int, int)
    + void evaporate(int)
}
note right
<i>App::evaporate</i> will be responsible of the evaporation process
<i>App::addPheromon</i> will be used to add pheromon on a selected cell
end note
class CELL {
    + int id
    + int state
    + float tau
}
interface ICellFunctor {
    + virtual void operator(const CELL)
}
interface IGrid {
    + virtual bool iInitialize()
    + virtual int iGetSizeX() const
    + virtual int iGetSizeY() const
    + virtual int iGetResolutionX() const
    + virtual int iGetResolutionY() const
    + virtual int iGetNumberOfCells() const
    + virtual bool iGetCellPosition(CELL, int&, int&) const
    + virtual bool iGetCellCoordinates(CELL, int&, int&) const
    + virtual bool iGetCellNumber(int, int, CELL&) const
    + virtual bool iGetContainingCell(const int, const int, CELL) const
    + virtual bool iIsWithinCell(const int, const int, CELL) const
    + virtual bool iAddObstacle(const CELL)
    + virtual bool iRemoveObstacle(const CELL)
    + virtual bool iIsObstacle(const CELL) const
    + virtual bool iAddPheromon(const CELL)
    + virtual void iUpdatePheromon(const CELL)
}
note right
<i>IGrid</i> defines <i>iAddPheromon</i>,
to add an amount of pheromon in a given CELL

and <i>iUpdatePheromon</i> to apply evaporation.
end note
class Grid implements IGrid 
IGrid ..> CELL : defines >
IGrid ..> ICellFunctor : defines >
ICellFunctor ..> CELL
Grid *--> CELL
App o--> IGrid : controls >
hide IGrid fields
hide CELL methods
hide Grid members
hide ICellFunctor members
@enduml
{% endplantuml %}

Basically, `IGrid::iUpdatePheromon` will update the amount of pheromons for each cell (`IGrid::CELL::pheromon`) by running the following formula:

$$ \tau_{ij}^{t} = \tau_{ij}^{t-1} \cdot (1 - \rho) + \Delta^{t}\tau_{ij} $$

Where :
- $$\tau_{ij}^{t}$$ is the *amount of pheromons in $$cell_{ij}$$ in the current timestep*
- $$\tau_{ij}^{t-1}$$ is the *amount of pheromons in $$cell_{ij}$$ in the previous timestep*
- $$\Delta^{t}\tau_{ij}$$ is the *amount of pheromon injected in the current timestep*
- $$\rho \in [0,1]$$ is the *evaporation coefficient*

As you might have guessed, this method will be called in `App::evaporate` at specific *time intervals*.

### Parameters
1. For numerical robustness, I will use the following global parameters in the implementation:
- $$P_{max}$$ : the *pheromon maximum capacity* of a cell
- $$P_{min}$$ : the *pheromon minimal capacity* of a cell (below this value, the amount of pheromon is set to $$0$$)
2. The interested reader can refer to the [source code][5] to check/set the value for parameters : $$\rho, P_{min}$$ and $$P_{max}$$

### PheromonViewer
To View *pheromons* and especially the *evaporation process*, I added a `PheromonViewer` that will be called in the `App::display` method, and basically applies an `IGrid::ICellFunctor` on every cell of the grid --- if their corresponding amount of pheromon is greater than zero -- to draw a red mark on the screen according to their current state (I use SFML::Color feature to define functor).

## Bundle
We are ready to instaciate our brand new `App` object with all the improvements for pheromon manipulation. Here is the **main.cpp** file :

{% highlight c++ %}
#include "App.h"
#include "GridViewer.h"
#include "ObstacleViewer.h"
#include "PheromonViewer.h"
#include "Grid.h"
#include <thread>
#include <SFML/Graphics.hpp>

#ifdef __linux__
#include <X11/Xlib.h>
#endif

int main()
{
#ifdef __linux__
    XInitThreads();
#endif

    // -- sfml windows
    sf::ContextSettings settings;
    settings.antialiasingLevel = 10;
    sf::RenderWindow window(
        sf::VideoMode(
            (App::DEFAULT_WIDTH*App::DEFAULT_RESX),
            (App::DEFAULT_HEIGHT*App::DEFAULT_RESY)
        ),
        "SFML 2D Grid",
        sf::Style::Titlebar | sf::Style::Close, 
        settings
    );
    window.clear(sf::Color::White);
    window.setFramerateLimit(120);
    window.setActive(false);
    
    // -- application
    App app;
    app.setWindow(&window);
    
    //-- grid 2D
    Grid g;
    g.setSizeX(App::DEFAULT_WIDTH);
    g.setSizeY(App::DEFAULT_HEIGHT);
    g.setResolutionX(App::DEFAULT_RESX);
    g.setResolutionY(App::DEFAULT_RESY);
    g.iInitialize();
    app.setGrid(&g);
    
    //-- viewer

    // grid lines
    GridViewer gviewer;
    gviewer.iActivate();
    
    // grid obstacles
    ObstacleViewer oviewer;
    oviewer.iActivate();

    // pheromons 
    PheromonViewer pviewer;
    pviewer.iActivate();

    // aggregator
    ViewerMgr mgr;
    mgr.iAddViewer(&oviewer);
    mgr.iAddViewer(&pviewer);
    mgr.iAddViewer(&gviewer);
    app.setViewer(&mgr);
    mgr.iActivate();

    // initialize gviewer (only after having attached it to the App object)
    gviewer.initialize();

    //-- launch application
    std::thread rendering_thread(&App::display, &app);
    std::thread evaporation_thread(&App::evaporate, &app);  
    app.run();
    rendering_thread.join();
    evaporation_thread.join();
    
    return 0;
}
{% endhighlight %}

The interested reader can fork the complete source code from [here][2] and run the following in a terminal at the project folder root :

{% highlight shell %}
  # on windows
  $ cmake  -G "Visual Studio 15 2017" -S . -B ./build 
  $ cmake  --build ./build --config Debug --target app
  $ ./bin/Debug/app

  # on linux
  $ mkdir build  
  $ cd build
  $ cmake -G "Unix Makefiles" .. -DCMAKE_BUILD_TYPE=Debug
  $ cmake --build ./ --target app
  $ ../bin/Debug/app
{% endhighlight %}

The program should display a clickable 2D Grid where the right-click adds an obstacle on the selected cell and the left-click removes it. With the mouse middle you should be able to drop pheromon on the grid. Once pheromons are dropped they automatically and smoothly start to evaporate.

![screenshot](/images/2d-grid-obstacles-pheromons.gif)

Enjoy and feel free to send me your feedbacks!

## More like this
{: .t60 }
{% include list-posts tag='sfml' %}

[1]: /modeling%20&%20simulation/sfml-2d-grid-obstacles/
[2]: https://fr.wikipedia.org/wiki/Stigmergie
[3]: http://homepages.ulb.ac.be/~jldeneub/images/Deneubourgetal1990.pdf
[4]: https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Clock.php
[5]: https://github.com/kanmeugne/sfml2dgrid/
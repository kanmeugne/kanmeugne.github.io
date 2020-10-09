---
layout: page-fullwidth
subheadline: "Modeling and Simulation"
title:  "2D-Grid with obstacles in Simulation"
teaser: "In a previous article, I shared a C++ code for grid manipulation. I presented an architecture that could be easily extended to include different type of controls and views. In this post, I am showing how easy it is to add obstacle and to keep track of them in the model."
tags:
    - sfml
    - cmake
    - c++
    - simulation
    - modeling
categories:
    - dev
image:
    thumb: 2D-Grid-obstacles-thumb.png
    caption_url: http://unsplash.com
---

As I mentioned in a [previous post][1], having a 2D-Grid alone is not really the point of this work. The source code I am sharing is meant to be used within a simulation framework where we need to model a trackable space. **Fig.1.** is a small modification of the architecture I presented in a previous post. Comparing to the previous architecture, some little changes have been made (see **Fig.2.**) on `IGrid` and `App` and have 3 new objects which are `ICellFunctor`, `ObstacleViewer` and `ViewerMgr`.

**Fig.1. The new Architecture with more methods in `App`, `IGrid` and 3 new objects : `ICellFunctor`, `ObstacleViewer` and `ViewerMgr`**
{% plantuml %}
@startuml
package geometry <<Frame>>
{
    class Point {
        + int x
        + int y
    }
    class Segment {
        + Point init
        + Point end
    }
    interface ISegmentFunctor {
        + virtual void operator(const Segment&)
    }
}
interface ICellFunctor {
    + virtual void operator(const CELL)
}
abstract class AbstractViewer {
    + virtual void iActivate()
    + virtual void iDeactivate()
    + virtual void iIsActive() const
    + virtual void iSetApp(App*)
    + virtual void iDisplay();
    # {abstract} virtual void iDraw()
    # bool _active = false
}
class App {
    ...
    .. obstacle-related actions ..
    + void addObstacle(int, int)
    + void removeObstacle(int, int)
}
class CELL {
    + int id
    + int state
}
class GridViewer extends AbstractViewer {
    - void drawLines(geometry::ISegmentFunctor&)
    + void initialize()
}
class ViewerMgr extends AbstractViewer {
    - vector<AbstractManager*> viewers
    + void iAddViewer(AbstractManager*)
}
class ObstacleViewer extends AbstractViewer {
    - drawObstacles(ICellFunctor&)
}
interface IGrid {
    ...
    .. functor broadcast ..
	+ virtual void iApplyOnCells(ICellFunctor&) const
}
class Grid implements IGrid {
    - int resx
    - int resy
    - int sizex
    - int sizey
}
IGrid ..> CELL : defines
Grid *-- CELL
App --> IGrid
App o-- AbstractViewer
Segment *-- Point
ISegmentFunctor ..> Segment
GridViewer ..> geometry
IGrid ..> ICellFunctor : defines
ICellFunctor ..> CELL
ObstacleViewer ..> ICellFunctor
hide members
{% endplantuml %}

## What's new ?

**Fig.2. (below) Details about the evolution of the architecture comparing to the previous one.**

{% plantuml %}
@startuml
interface ICellFunctor {
    + virtual void operator(const CELL)
}
abstract class AbstractViewer {
    + virtual void iActivate()
    + virtual void iDeactivate()
    + virtual void iIsActive() const
    + virtual void iSetApp(App*)
    + virtual void iDisplay();
    # {abstract} virtual void iDraw()
    # bool _active = false
}
class App {
    ...
    .. obstacle-related actions ..
    + void addObstacle(int, int)
    + void removeObstacle(int, int)
}
class CELL {
    + int id
    + int state
}
class ViewerMgr extends AbstractViewer {
    - vector<AbstractManager*> viewers
    + void iAddViewer(AbstractManager*)
}
class ObstacleViewer extends AbstractViewer {
    - drawObstacles(ICellFunctor&)
}
interface IGrid {
    ...
    .. functor broadcast ..
	+ virtual void iApplyOnCells(ICellFunctor&) const
}
IGrid ..> CELL : defines
App --> IGrid
App o-- AbstractViewer
IGrid ..> ICellFunctor : defines
ICellFunctor ..> CELL
ObstacleViewer ..> ICellFunctor

hide App fields
hide AbstractViewer members
hide CELL members
hide ICellFunctor fields
hide ObstacleViewer fields
@enduml
{% endplantuml %}

### App

The `App` object defines two more methods in order to handle obstacle-related actions. Here, I will use mouse events to add and remove obstacle on the grid attached to the `App`. As mentionned in the [first post of this serie][2], the logic will be held by the `App::run` method.

Below, an extract of the implementation of `App::run` (more details [here][3]). Keep in mind that the `App::run` method will add obstacles on the right-click and remove obstacle on the left-click.

{% highlight c++ %}
void App::run()
{
    _window->setActive(false);
    while (_window->isOpen())
    {
        sf::Event event;
        while (_window->pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                _window->close();
            if (event.type == sf::Event::MouseButtonPressed)
            {
                if (event.mouseButton.button == sf::Mouse::Right)
                {
                    int posx = event.mouseButton.x;
                    int posy = event.mouseButton.y;
                    printf("Adding obstacle at position [x=%d, y=%d] ", posx, posy);
                    printf(" ... %s!\n", addObstacle(posx, posy)?"SUCCESSFUL":"FAILED");
                }
                if (event.mouseButton.button == sf::Mouse::Left)
                {
                    int posx = event.mouseButton.x;
                    int posy = event.mouseButton.y;
                    printf("Removing obstacle at position [x=%d, y=%d] ", posx, posy);
                    printf(" ... %s!\n", removeObstacle(posx, posy)?"SUCCESSFUL":"FAILED");
                }
            }
            if (event.type == sf::Event::MouseMoved)
            {   
            }
        }
    }
}
{% endhighlight %}
Considering the location checks methods properly defined in IGrid, the implemenation of `App::addObstacle` and `App::removeObstacle` is straightforward. Our obstacle model is very simple thanks to the grid approach for the space representation : adding or removing an obstacle in the space is equivalent to changing the state value of one or several cells !

{% highlight c++ %}
bool App::addObstacle(int posx, int posy)
{
    IGrid::CELL cell;
    int resx = getGrid()->iGetResolutionX();
    int resy = getGrid()->iGetResolutionY();
    bool thereisacell = getGrid()->iGetCellNumber(posy/resy, posx/resx, cell);
    if (thereisacell)
        printf("(CellNo: %d)", cell);
        
    return thereisacell && (getGrid()->iAddObstacle(cell));
}

bool App::removeObstacle (int posx, int posy)
{
    IGrid::CELL cell;
    int resx = getGrid()->iGetResolutionX();
    int resy = getGrid()->iGetResolutionY();
    bool thereisacell = getGrid()->iGetCellNumber(posy/resy, posx/resx, cell);
    if (thereisacell)
        printf("(CellNo: %d)", cell);
    return thereisacell && (getGrid()->iRemoveObstacle(cell));
}
{% endhighlight %}


### Other objects : IGrid, ICellFunctor, ObstacleViewer and ViewerMgr

`IGrid` defines one more method called `IGrid::iApplyOnCells` which takes a functor on cells - `IGrid::ICellFunctor` - as parameter and applies it on every cell of the grid. 

`ObstacleViewer` is in charge of displaying the obstacle and should define an `ICellFunctor` for that purpose (that explains the relation on the Fig.1.).

`ViewerMgr` is a special `AbstractViewer` that agregates (cf. Composite pattern) other `AbstractViewer`'s with the method `ViewerMgr::iAddViewer`. I introduce this object in order to separate view concerns and to be able to activate several views at the same time without complicating the relationship between `App` and `AbstractViewer`.

## Bundle

At the end, we have an even better architecture since we can manipulate several views in a simple way. The way we deal with obstacle very straightforward - there is an obstacle where `CELL`'s mask is set to `1`. With all this settings we can finally get everything together. The **main.cpp** file looks like below.

{% highlight c++ %}
#include "App.h"
#include "GridViewer.h"
#include "ObstacleViewer.h"
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
    
    // aggregator
    ViewerMgr mgr;
    mgr.iAddViewer(&oviewer);
    mgr.iAddViewer(&gviewer);
    app.setViewer(&mgr);
    mgr.iActivate();
    
    // initialize gviewer (only after having attached it to the App object)
    gviewer.initialize();

    //-- launch application
    std::thread rendering_thread(&App::display, &app);
    app.run();
    rendering_thread.join();
    
    return 0;
}
{% endhighlight %}

The interested reader can fork the complete source code from [here][3] and run the following in a terminal at the project folder root :

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

The program should display a clickable 2D Grid where the right-click adds an obstacle on the selected cell and the left-click removes it.

![screenshot](/images/2d-grid-demo.png)

Enjoy and feel free to send me your feedbacks!

## More like this
{: .t60 }
{% include list-posts tag='sfml' %}

[1]: https://github.com/kanmeugne/sfml2dgrid
[2]: https://www.sfml-dev.org/tutorials/2.5/compile-with-cmake.php
[3]: https://crascit.com/2015/07/25/cmake-gtest/
[4]: https://github.com/google/googletest
[5]: https://cmake.org/
[6]: https://www.sfml-dev.org/documentation/2.5.1/
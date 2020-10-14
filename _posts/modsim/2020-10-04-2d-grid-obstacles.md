---
layout: page-fullwidth
title:  "2D-Grid with obstacles in Simulation"
teaser: "In a previous article, I shared a C++ project for grid manipulation. I presented an architecture that could be easily extended to include different types of controls and views. In this post, I am showing a way it is to add obstacles manipulation to the porject and to keep track of them."
tags:
    - sfml
    - cmake
    - c++
    - simulation
    - modeling
categories:
    - modeling & simulation
image:
    thumb: 2D-Grid-obstacles-thumb.png
---

As I mentioned in a [previous post][1], having a 2D-Grid alone is not really the point of this work. The source code I am sharing is meant to be used within a simulation framework, where we need to model a trackable space. With that in mind, I am going to extend the original architecture in order to add obstacle manipulation, i.e. the possibility to add and remove obstacles in our 2D-Grid environment with some given controls. 

{% plantuml %}
@startuml
title: <size:20>Fig. 1. Updated Architecture (with obstacle manipulation features)</size>
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
Grid *--> CELL : is composed of
App o--> IGrid : manipulates
App o-- AbstractViewer
Segment *--> Point : is bounded by
ISegmentFunctor ..> Segment : uses
GridViewer ..> geometry : uses
IGrid ..> ICellFunctor : defines
ICellFunctor ..> CELL : uses
ObstacleViewer ..> ICellFunctor : runs
hide members
{% endplantuml %}

For that purpose, I have made some improvements on the [previous architecture][1], as you can see in **Fig. 1**. Briefly, I have updated 3 existing objects --- `App`, `IGrid` and `Grid` --- and created 3 new objects --- `ObstacleViewer`, `ICellFunctor` and `ViewerMgr`. More details, about the improvements below.

## The App Object

The App object is augmented with two more methods responsible of adding and removing obstacles in the 2D Grid respectively. We will discuss the storage model for obstacle management in the next section. 

{% plantuml %}
@startuml
title: <size:20>Fig. 2. App Object</size>
class App {
    + {static} const int DEFAULT_HEIGHT
    + {static} const int DEFAULT_WIDTH
    + {static} const int DEFAULT_RESX
    + {static} const int DEFAULT_RESY
    + void run()
    + void display()
    + bool addObstacle(int, int)
    + bool removeObstacle(int, int)
}
abstract class AbstractViewer
interface IGrid
App o-- AbstractViewer
App o--> IGrid
hide IGrid members
hide AbstractViewer members

@enduml
{% endplantuml %}

## What's new ?

_**Fig.2. (below) Details about the evolution of the architecture comparing to the previous one.**_

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

The `App` object defines two *methods* in order to handle obstacle-related actions : `App::addObstacles` and App::`App::removeObstacles`. Both methods will be called in the `App::run` method (see implementation below).

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
Our obstacle model is very simple thanks to the grid approach for the space representation. In fact, adding or removing an obstacle in the space is equivalent to changing the state value of one or several cells (from 0 to 1 and vice versa). So, considering the location checks methods properly defined in `IGrid`, the implemenation of `App::addObstacle` and `App::removeObstacle` is straightforward.

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


### Viewers

`IGrid` defines one more method called `IGrid::iApplyOnCells` which takes a functor on cells - `IGrid::ICellFunctor` - as the only parameter and applies it on every cell of the grid. This methods will be called in `ObstacleViewer::drawObstacles` method, in charge of displaying the obstacles of the grid.

`ViewerMgr` is a special `AbstractViewer` that agregates (cf. Composite pattern) other `AbstractViewer`'s with the method `ViewerMgr::iAddViewer`. I introduce this object in order to separate view concerns and to be able to activate several views at the same time without complicating the relationship between `App` and `AbstractViewer`.

## Bundle

We have an even better architecture since we can manipulate several views in a simple way. The main file below shows how everything gets together.

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

The program should display a clickable 2D Grid where the right-click adds an obstacle on the selected cell and the left-click removes it.

![screenshot](/images/2d-grid-obstacles.gif)

Enjoy and feel free to send me your feedbacks!

## More like this
{: .t60 }
{% include list-posts tag='sfml' %}

[1]: /modeling%20&%20simulation/sfml-2d-grid/
[2]: https://github.com/kanmeugne/sfml2dgrid/releases/tag/sfml-2d-obstacles-grid
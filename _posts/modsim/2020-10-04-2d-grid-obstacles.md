---
layout: page-fullwidth
title:  "2D-Grid with obstacles in Simulation"
teaser: "In a previous article, I shared a C++ project for 2D-grid manipulation. I presented an architecture that could be easily extended to include different types of controls and views. In this post, I am showing a way it is to add obstacles manipulation to the porject and to keep track of them."
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
title: <size:10>Fig. 1. Updated Architecture with obstacle manipulation features and better integration of viewers</size>
package geometry <<Frame>>
{
    class Point 
    class Segment 
    interface ISegmentFunctor 
}
IGrid <.. CELL : subtype <
Grid *--> CELL 
App o-> IGrid : controls >
App o-- AbstractViewer
ViewerMgr o--> AbstractViewer
Segment *-> Point : has 2 >
ISegmentFunctor .> Segment
GridViewer ..> geometry
IGrid <.. ICellFunctor : subtype <
ICellFunctor ..> CELL
ObstacleViewer ..> ICellFunctor : runs >
hide members
{% endplantuml %}

For that purpose, I have made some improvements on the [previous architecture][1], as you can see in **Fig. 1**.

Briefly, I have updated 3 existing objects --- *App*, *IGrid* and *Grid* --- and created 3 new objects --- *ObstacleViewer*, *IGrid::ICellFunctor* and *ViewerMgr*. More details below.

## App

The *App* object is augmented with *App::addObstacle* and *App::removeObstacle* both responsible of *adding* and *removing* obstacles in the 2D Grid respectively (see Fig. 2). To keep things simple, an obstacle is represented as a non-zero value in a cell --- so *App::addObstacle* and *App::removeObstacle* effect will be to set the value of a given *IGrid::CELL* object (selected by the mouse click).
{% plantuml %}
@startuml
title: <size:10>Fig. 2. App Object (with <i>addObstacle</i> and <i>removeObstacle</i>)</size>
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

note right
App object has 2 more methods
# App::addObstacle
# App::removeObstacle
end note
abstract class AbstractViewer
interface IGrid
App o-- AbstractViewer
App o--> IGrid : controls >
hide IGrid members
hide AbstractViewer members

@enduml
{% endplantuml %}

## IGrid

The *IGrid* interface is augmented with 3 more methods --- *IGrid::iAddObstacle*, *IGrid::iRemoveObstacle* and *IGrid::iIsObstacle* --- necessary to edit the status of a *IGrid::CELL* status (*IGrid::iAddObstacle* and *IGrid::iRemoveObstacle*) and to check whether a given *IGrid::CELL* is an obstacle or not (*IGrid::iIsObstacle*). *IGrid* defines one more method called *IGrid::iApplyOnCells* which takes a functor on cells - *IGrid::ICellFunctor* - as the only parameter and applies it on every cell of the grid. For the record, this method is called in *ObstacleViewer::drawObstacles* method (see next section), in charge of displaying the obstacles of the grid. **Fig. 3** gives extensive details about the *IGrid* new look and its relations with other classes definitions.

{% plantuml %}
@startuml
title: <size:10>Fig. 3. Evolution of the IGrid interface, with 3 more methods : <i>iAddObstacle</i>, <i>iRemoveObstacle</i> and <i>iIsObstacle</i></size>
class App {
    + {static} const int DEFAULT_HEIGHT
    + {static} const int DEFAULT_WIDTH
    + {static} const int DEFAULT_RESX
    + {static} const int DEFAULT_RESY
    + void run()
    + void display()
}
class CELL {
    + int id
    + int state
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
}
note right
IGrid defines 3 more methods :
# IGrid::iAddObstacle : add an obstacle in the grid
# IGrid::iRemoveObstacle : remove an obstacle from the grid
# IGrid::iIsObstacle : to check whether a cell is an obstacle or not
end note
class Grid implements IGrid {
    - int resx
    - int resy
    - int sizex
    - int sizey
}
IGrid <.. CELL : subtype <
IGrid <.. ICellFunctor : subtype <
ICellFunctor ..> CELL
Grid *--> CELL
App o--> IGrid : controls >
hide App members
hide IGrid fields
hide CELL methods
hide Grid methods
hide ICellFunctor fields
@enduml
{% endplantuml %}

## ViewerMgr and ObstacleViewer

*ViewerMgr* is a special *AbstractViewer* that agregates (cf. Composite pattern) other *AbstractViewer* with the method *ViewerMgr::iAddViewer*. I introduce this object in order to separate view concerns and to be able to activate several views at the same time without complicating the relationship between *App* and *AbstractViewer*. We will use this *meta* viewer to attach en *ObstacleViewer* to the App in order to display the grid lines and the obstacles at the same time.

{% plantuml %}
@startuml
title
<size:10> Fig. 4. ViewerMgr is a meta viewer that agregates more than one viewer. It will be used to add a viewer for obstacle (ObstacleViewer) next to the viewer for </size>
<size:10> lines (GridViewer) without changing the relationship between App and AbstractObject</size>
end title
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
class ObstacleViewer extends AbstractViewer {
    - drawObstacles(ICellFunctor&)
}
class ViewerMgr extends AbstractViewer {
    + void iAddViewer(AbstractManager*)
}
ViewerMgr o--> AbstractViewer
ObstacleViewer ..> ICellFunctor : runs >
hide ObstacleViewer field
hide ICellFunctor field
hide ViewerMgr field
{% endplantuml %}

## Bundle

We are ready to instaciate our new App object with all the improvements. The **main.cpp** for the 2D grid demo App with obstacle management is the following :

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
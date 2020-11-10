---
title:  "2D Grid with obstacles"
teaser: "Building a 2D Grid is a very common exercice in computer vision, fluid simulation, navigation flow simulation, etc. In this post, I am sharing a C++ code for 2D Grid manipulation with obstacle management."
tags:
    - sfml
    - cmake
    - c++
    - simulation
    - modeling
categories:
    - modeling & simulation
---

Using a regular 2D Grid to model the navigable space is a good choice if you want to simulate moving agents (ex: vehicules, pedestrians). In fact, 2D Grids can be seen as partitions of the space and, therefore, provide an excellent framework for path planning and collision avoidance algorithms deployment. Moreover, by adding state variables to grid cells, we end up with a very affordable way to manage obstacles and other kind of semantics in the space. 

In this post, I am upgrading an existing *object oriented architecture* that [I shared recently][1] as a starting point for those who wanted to have a 2D Grid in their simulation app. Back then, the provided features were limited to grid dimension setting and visualization. In this new version, I am adding a simple obstacle management by attaching state variables to grid cells --- a complete implementation in C++ is also provided for demonstration.


{% plantuml %}
@startuml
header: <size:10> <font color=blue>Fig. 1.</font> Architecture of our 2D Grid App </size>

class App {}

package env 
{
    
}

package viewers
{
}

package geometry
{
}

App ..> viewers
App ..> env
viewers ..> geometry

hide members
@enduml

{% endplantuml %}

```terminal
sfml2dgrid
.
├── CMakeLists.txt
├── deps
│   └── sfml
│       └── CMakeLists.txt.in
└── sfml2dgrid
    ├── CMakeLists.txt
    ├── main.cpp
    ├── app
    │   ├── include
    │   │   └── App.h
    │   └── src
    │       └── App.cpp
    ├── env
    │   ├── include
    │   │   ├── Grid.h
    │   │   └── IGrid.h
    │   └── src
    │       └── Grid.cpp
    ├── geometry
    │   └── include
    │       └── geometry.h
    └── viewers
        ├── include
        │   ├── AbstractViewer.h
        │   └── GridViewer.h
        └── src
            ├── AbstractViewer.cpp
            └── GridViewer.cpp
```
> **Note**: the file tree of the project with the source (.cpp) and header (.h) files. I am just going to discuss about the upgrade that I made from the previous version.

Comparing to the [previous version][1], I have updated 3 existing objects --- *App*, *env::IGrid* and *env::Grid* --- and created 3 new objects --- *viewers::ObstacleViewer*, *env::ICellFunctor* and *viewers::ViewerMgr*. More details below.

## App

The *App* object is augmented with *App::addObstacle* and *App::removeObstacle* both responsible of *adding* and *removing* obstacles in the 2D Grid respectively (see Fig. 2). As I teased in the introdution, state variables are associated to grid cells in order to store occupancy information --- this is how the model handle obstacles : if a cell occupied, it is considered as an obstacle.

{% plantuml %}
@startuml
header: <font color=blue>Fig. 2.</font> App Object (with <i>addObstacle</i> and <i>removeObstacle</i>)
scale 0.9
class App {
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

**App.h**

```c++
#ifndef APP_H
#define APP_H

namespace sf
{
	class RenderWindow;
};

namespace env
{
	class IGrid;
};

namespace viewers
{
	class AbstractViewer;
};

class App
{
private:
	// sfml render window
	sf::RenderWindow *_window = nullptr;
	// the 2D grid pointer
	env::IGrid *_grid = nullptr;
	// a pointer to the viewer
	// this could be a set of viewer actually
	// if we consider component behavior
	viewers::AbstractViewer *_viewer = nullptr;

public:
	// theorical width of the environment
	// will match the grid width in terms of number of cells.
	static const int DEFAULT_WIDTH;
	// theorical height of the environment.
	static const int DEFAULT_HEIGHT;
	// x-resolution of the grid i.e. the x-size of a cell
	static const int DEFAULT_RESX;
	// y-resolution of the grid i.e. the y-size of a cell
	static const int DEFAULT_RESY;
	// attach window to the app
	void setWindow(sf::RenderWindow *);
	// attach a specific viewer
	void setViewer(viewers::AbstractViewer *);
	// attach a grid (should have been initialized)
	void setGrid(env::IGrid *);
	// return the attached grid
	env::IGrid *getGrid();
	// return the attached window
	sf::RenderWindow *getWindow();
	// run the application (the logic)
	void run();
	// show content (display routines)
	void display();
	// add obstacle control
	bool addObstacle(int, int);
	// remove obstacle control
	bool removeObstacle (int, int);

	App() = default;
	virtual ~App();
};
#endif // !APP_H
```

## IGrid

The *IGrid* interface is augmented with 3 more methods --- *IGrid::iAddObstacle*, *IGrid::iRemoveObstacle* and *IGrid::iIsObstacle* --- necessary to edit the state of a *CELL* status (*IGrid::iAddObstacle* and *IGrid::iRemoveObstacle*) and to check whether a given *CELL* is an obstacle or not (*IGrid::iIsObstacle*). *IGrid* defines one more method called *IGrid::iApplyOnCells* which takes a functor on cells - *env::ICellFunctor* - as the only parameter and applies it on every cell of the grid. For the record, this method is called in *ObstacleViewer::drawObstacles* method (see next section), in charge of displaying the obstacles of the grid. **Fig. 3** gives extensive details about the *IGrid* new look and its relations with other classes definitions.

{% plantuml %}
@startuml
header
<font color=blue>Fig. 3.</font> Evolution of the IGrid interface, with 3 more methods :
<i>iAddObstacle</i>, <i>iRemoveObstacle</i> and <i>iIsObstacle</i>
end header
scale 0.8
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
    + virtual bool iGetCellLocation(CELL, int&, int&) const
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
header
<font color=blue>Fig. 4.</font> ViewerMgr is a meta viewer that agregates more than one viewer.
It will be used to add a viewer for obstacle (ObstacleViewer) next to the viewer for
lines (GridViewer) without changing the relationship between App and AbstractObject
end header
scale 0.9
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


[1]: /modeling%20&%20simulation/sfml-2d-grid/
[2]: https://github.com/kanmeugne/sfml2dgrid/releases/tag/sfml-2d-obstacles-grid


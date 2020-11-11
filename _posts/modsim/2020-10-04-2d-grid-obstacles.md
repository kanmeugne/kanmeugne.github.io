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
        │   ├── GridViewer.h
        │   └── ObstacleViewer.h
        └── src
            ├── AbstractViewer.cpp
            ├── GridViewer.cpp
            └── ObstacleViewer.cpp
```
> **Note**: the file tree of the project with the source (.cpp) and header (.h) files. I am just going to discuss about the upgrade that I made from the previous version.

Comparing to the [previous version][1], I have updated 3 existing objects --- *App*, *env::IGrid* and *env::Grid* --- and created 3 new objects --- *viewers::ObstacleViewer*, *env::ICellFunctor* and *viewers::ViewerMgr*. More details below.

## App

The *App* object is augmented with *App::addObstacle* and *App::removeObstacle* both responsible of *adding* and *removing* obstacles in the 2D Grid respectively (see Fig. 2). As I teased in the introdution, state variables are associated to grid cells in order to store occupancy information --- this is how the model handle obstacles : if a cell occupied, it is considered as an obstacle.


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

The *IGrid* interface is augmented with 3 obstacle-related methods --- *IGrid::iAddObstacle*, *IGrid::iRemoveObstacle* and *IGrid::iIsObstacle* --- necessary to edit the state of a *CELL* status (*IGrid::iAddObstacle* and *IGrid::iRemoveObstacle*) and to check whether a given *CELL* is an obstacle or not (*IGrid::iIsObstacle*).

*IGrid* defines one more method called *IGrid::iApplyOnCells* which takes a functor on cells --- *env::ICellFunctor* --- as the only parameter and applies it on every cell of the grid. For the record, this method is called in *ObstacleViewer::drawObstacles* method (see next section), in charge of displaying the obstacles of the grid. **Fig. 3** gives extensive details about the *IGrid* new look and its relations with other classes definitions.


**IGrid.h**
```c++
#ifndef IGRID_H
#define IGRID_H

namespace env
{
    struct CELL
    {
        int _id; // id of the cell
        bool _mask;
        CELL() = default;
        CELL(const CELL &) = default;
    };
    // Functor definition to apply on cell
    // We can inherit from this to function
    // to apply on cells
    class ICellFunctor
    {
    public:
        virtual void operator()(
            const CELL & // cell_id
            ) = 0;
    };
    // IGrid
    class IGrid
    {
    public:
        virtual ~IGrid() = default;
        // returns the width
        virtual int iGetSizeX() const = 0;
        // returns the height
        virtual int iGetSizeY() const = 0;
        // returns the number of cells in the grid
        virtual int iGetNumberOfCells() const = 0;
        // gets the width of a cell (in terms of pixels)
        virtual int iGetResolutionX() const = 0;
        // gets the height of a cell (in terms of pixels)
        virtual int iGetResolutionY() const = 0;
        // applies functor on Cells
        virtual void iApplyOnCells(ICellFunctor &) const = 0;
        //-- Test
        // relative position of a cell according to its id
        virtual bool iGetCellPosition(
            const CELL &, // cell
            int &,        // posx
            int &         // posy
        ) const = 0;
        // coordinates of a cell accoring to its id
        virtual bool iGetCellCoordinates(
            const CELL &, // cell
            int &,        // row_number
            int &         // column_number
        ) const = 0;
        // cell rank of the the cell according
        // to its relative position in the grid
        virtual bool iGetCellNumber(
            int, // row_number
            int, // column_number
            CELL &) const = 0;
        // the containing cell given the coordinates in the 2D space
        virtual bool iGetContainingCell(
            int,   // posx
            int,   // posy
            CELL & // cell
        ) const = 0;
        // checks if a given point is within a given cell
        virtual bool iIsWithinCell(
            int,         // posx
            int,         // posy
            const CELL & // cell
        ) const = 0;
        // initializes the vector of cells, obstacle mask, etc.
        virtual void iInitialize() = 0;
        // add obstacle to the grid
        virtual bool iAddObstacle(const CELL &) = 0;
        // remove obstacle from the grid
        virtual bool iRemoveObstacle(const CELL &) = 0;
        // return the obstacle status : true if obstacle, false otherwise
        virtual bool iIsObstacle(const CELL &) const = 0;
    };
} // namespace env
#endif // !IGRID_H

```

## ViewerMgr and ObstacleViewer

*ViewerMgr* is a special *AbstractViewer* that agregates (cf. Composite pattern) other *AbstractViewer* with the method *ViewerMgr::iAddViewer*. I introduce this pattern in order to separate view concerns and to be able to activate several views at the same time. We will use this *meta* viewer to attach an *ObstacleViewer* to the App in order to display the grid lines and the obstacles at the same time.

**AbstractViewer.h**
```c++
#ifndef ABSTRACTVIEWER_H
#define ABSTRACTVIEWER_H
#include <vector>

class App;

namespace viewers
{
	// AbstractViewer
	class AbstractViewer
	{
	public:
		// activate the viewer. If activated, it provides the desired view
		virtual void iActivate();

		// deactivate the viewer. Do not display anaything if deactivated
		virtual void iDeactivate();

		// return True if the viewer is active
		virtual bool iIsActive() const;

		// display function
		virtual void iDisplay();

		// attach the application object
		virtual void iSetApp(App *);

		virtual ~AbstractViewer() = default;
		AbstractViewer() = default;

	protected:
		// specific draw method (to be concretized in child classes)
		virtual void iDraw() = 0;
		bool _active = false;
		App *_app;
	};

	// viewer manager, using the composite pattern to
	// aggregate several viewers into one
	class ViewerMgr : public AbstractViewer
	{
	public:
		virtual void iAddViewer(AbstractViewer *);
		virtual ~ViewerMgr() = default;
		ViewerMgr() = default;
		virtual void iSetApp(App *) override;

	protected:
		virtual void iDraw();

	private:
		std::vector<AbstractViewer *> _viewers;
	};
} // namespace viewers

#endif // ABSTRACTVIEWER_H
```

**ObstacleViewer.h**
```c++
#ifndef OBSTACLEVIEWER_H
#define OBSTACLEVIEWER_H

#include "AbstractViewer.h"

namespace env
{
    class ICellFunctor;
};

namespace viewers
{
    class ObstacleViewer : public AbstractViewer
    {
    public:
        ObstacleViewer() = default;
        virtual ~ObstacleViewer() = default;

    protected:
        virtual void iDraw();

    private:
        void drawObstacles(env::ICellFunctor &);
    };
} // namespace viewers
#endif // !OBSTACLEVIEWER_H
```
## Demo

We are ready to run our brand new App with all the improvements. The **main.cpp** for the 2D grid demo App with obstacle management is exactly the following :

```c++
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
        "SFML 2D Grid with obstacles",
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
    env::Grid g;
    g.setSizeX(App::DEFAULT_WIDTH);
    g.setSizeY(App::DEFAULT_HEIGHT);
    g.setResolutionX(App::DEFAULT_RESX);
    g.setResolutionY(App::DEFAULT_RESY);
    g.iInitialize();
    app.setGrid(&g);

    //-- viewer
    viewers::GridViewer gviewer;
    gviewer.iActivate();

    // grid obstacles
    viewers::ObstacleViewer oviewer;
    oviewer.iActivate();

    // aggregator
    viewers::ViewerMgr mgr;
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
```

The interested reader can fork the complete source code from [here][2] and run the following in a terminal at the root of the project folder :

### On windows

```shell
  $ cmake  -G "Visual Studio 15 2017" -S . -B ./build 
  $ cmake  --build ./build --config Debug --target app
  $ ./bin/Debug/app
```
### On linux

```shell
  $ mkdir build  
  $ cd build
  $ cmake -G "Unix Makefiles" .. -DCMAKE_BUILD_TYPE=Debug
  $ cmake --build ./ --target app
  $ ../bin/Debug/app
```

The program should display a clickable 2D Grid where the right-click adds an obstacle on the selected cell and the left-click removes it.

![screenshot](/images/sfml-2d-grid-obstacles.gif)

Enjoy and feel free to send me your feedbacks!


[1]: /modeling%20&%20simulation/sfml-2d-grid/
[2]: https://github.com/kanmeugne/sfml2dgrid/releases/tag/sfml-2d-obstacles-grid


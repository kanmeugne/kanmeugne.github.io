---
title:  "Pheromon evaporation on a 2D Grid"
teaser: "In a previous article, I presented a C++ SFML application that models a 2D Grid with the possibility to add and remove obstacles. In this post, I am going to add a very nice feature : <i>pheromon evaporation</i>. I will discuss how this kind of feature could profit operational research and online optimization modelers."
tags:
    - sfml
    - cmake
    - c++
    - modeling
    - pheromon
    - evaporation
    - simulation
categories:
    - modeling & simulation
math: true
---


You should now be familiar with our [2D Grid app][6].

In a [previous post][1], I updated its original object-oriented architecture in order to implement an affordable obstacle feature. Here, I am going to upgrade the architecture again in order to implement a pheromon evaporation feature.

You can see pheromon like a chemical substance that is dropped somewhere --- as a mark of an organic activity --- and then evaporates overtime. Pheromon paradigm is a very productive way to implement a *stigmergic* behavior --- which is a type of behavior based on indirect communication through a common and shared space. Stigmergy explains the emergence of collective behavior among several social species with limited intellectual abilities. The concept has been first introduced by the french biologist [Pierre-Paul Grassé][2] and systematically studied by [Deneubourg][3] for different ants species.

The proposed improvements of the previous [object-oriented architecture][1] will focus on mimicking and illustrating an ant-like pheromon evaporation. Simply put, I am going to do the following : 

1. add another viewer for pheromon --- *PheromonViewer* --- and more controls in the *App* object -- *App::addPheromon*
2. upgrade *IGrid*, and consequently *Grid*,  to declare and implement pheromon related methods
3. define a new method --- *App::evaporate* --- responsible of the evaporation process.

![Fig. 1.](http://www.plantuml.com/plantuml/svg/JOzHJe0m38RVUugUO8YFP2wG63t7s0eiRenq6qQ9TvSfKVhsV_tx9UrJrfnByB2GjiGTokG-gMiV5CefhXbDz96VHbz8lRXPO2jaKnSA1pQBosHoOS8uOIoCZy_uoixYi6qecwfI0CRhb7xGyGMmQVEcPq9QTuG3y1ouuTtHEt62umvdYe4oV_pIQX92_LMlN8rmiPh_hs1nbZ1G66kzfzMU7ty3)


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
    │   ├── include
    │   │   ├── App.h
    │   │   └── dynamics.h
    │   └── src
    │       └── App.cpp
    ├── env
    │   ├── include
    │   │   ├── Grid.h
    │   │   └── IGrid.h
    │   └── src
    │       └── Grid.cpp
    ├── geometry
    │   ├── include
    │   │   └── geometry.h
    │   └── src
    └── viewers
        ├── include
        │   ├── AbstractViewer.h
        │   ├── GridViewer.h
        │   ├── ObstacleViewer.h
        │   └── PheromonViewer.h
        └── src
            ├── AbstractViewer.cpp
            ├── GridViewer.cpp
            ├── ObstacleViewer.cpp
            └── PheromonViewer.cpp

```
> **Note**: the file tree of the project with the source (.cpp) and header (.h) files. I am just going to discuss about the upgrade that I made from the previous version.

## Pheromon modeling and evaporation

The *App* object is augmented with *App::addPheromon* and *App::evaporate* methods both responsible of *adding* a little amount of pheromon in a selected cell, and *evaporating* pheromons over time --- see Fig. 2. 

![Fig. 2.](http://www.plantuml.com/plantuml/svg/bPJVRzCm4CVV_LVS4-fWwmPlL5MqgM9JfObDGuZnwCRNzeJZY-nBqORutx7TBBYq555LINptkH-sxklB8DfBrrWLQcqHBiRlbP1OdC0qq3DEtbvKrxGQmxinNBKjQ6TWVkF903Mjvmqsw2HCpnDRgThg43BtGq7ylO4DHzHtRdHI6Wo5rkgdmRXWjg2DkLi4qRN54JavXVZO0JmsiU0n9YUvNwFdXbq15ALnerlsMd2KFJ-LOq7mj5gBcb8LUvzC1cPwJXLy9sjX4HaCBRj02ul0Iv0r3ic87RIUQmnXI59qifEc2sX0E7KARGz0_6i8QB6MwAtHMeNEGEgjFzJPXzjRc4xHRQfgE5ro0cJA_JSwVElNIyjQG7GNDnb1z4jT8yndQErrvsfXlvTlGruwRRSdnIrw7QbHpIv8Rk2aJDNFGPbZQ8FcZeIqfMSSdPJDtg1yYiR7WzQlUzO736oxTBwFHtpxSH-xPe7-Rfas6tR6BBkJ-Pu3fU1YUw_okFJFWq4pPc_8QS7mNt7RVeQ9FCrX1r1seicHMwMWlaBcXy-3eVFmXMHDxaXK_ZmKUrN8xg_Gmo2zuzmy_9kvAYHQGgLAwNDhjE0XRb-pUGwpnWmkoM60Q2pB9D-fwZMd7UY6EoT9n8C0ALfXHHjqkOfIsfcSPhUHN2KbQblxLCf__ATUN-v8kxqXUx6exMiyhdAPObdgKfMBtt1cNv-TdVNhTCzoMkKiQaq6-p9BGcj2jcGK6fGrlrWoqc2QoDvIrjlrf1fzjajqfcliBm00)

*App::evaporate* will take a *time interval* as parameter in order to schedule the *evaporation process* --- I use [SFML *clocks*][4] to implement this.

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
	// evaporation cycle
	void evaporate();
	// add obstacle control
	bool addObstacle(int, int);
	// remove obstacle control
	bool removeObstacle(int, int);
	// add Pheromon
	bool addPheromon(int, int);

	App() = default;
	virtual ~App();
};
#endif // !APP_H
```

A special method to apply evaporation is also defined in *IGrid* --- *IGrid::iUpdatePheromon*.

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
        float _tau; // amount of pheromon
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
        // pheromons
        virtual bool iAddPheromon(
            const CELL &,// cell no
            const float  // pheromon deposit
        ) = 0;
        virtual void iUpdatePheromon(const int&) = 0;
    };
} // namespace env
#endif // !IGRID_H
```

Basically, *IGrid::iUpdatePheromon* will update the amount of pheromons for each cell --- *CELL::tau* --- by running the following formula:

$$ \tau_{ij}^{t} = \tau_{ij}^{t-1} \cdot (1 - \rho) + \Delta^{t}\tau_{ij} $$

Where :
- $$\tau_{ij}^{t}$$ is the *amount of pheromons in $$cell_{ij}$$ in the current timestep*
- $$\tau_{ij}^{t-1}$$ is the *amount of pheromons in $$cell_{ij}$$ in the previous timestep*
- $$\Delta^{t}\tau_{ij}$$ is the *amount of pheromon injected in the current timestep*
- $$\rho \in [0,1]$$ is the *evaporation coefficient*

As you might have guessed, this method will be called in *App::evaporate* at specific *time intervals*.

## Parameters

For numerical robustness, I will use the following global parameters in the implementation:
- $$P_{max}$$ : the *pheromon maximum capacity* of a cell
- $$P_{min}$$ : the *pheromon minimal capacity* of a cell (below this value, the amount of pheromon is set to $$0$$)

The interested reader can refer to the [source code][5] to check/set the value for parameters : $$\rho, P_{min}$$ and $$P_{max}$$

## PheromonViewer

To visualize *pheromons* and especially the *evaporation process*, I added a *PheromonViewer* that will be called in the *App::display* method. *PheromonViewer::iDraw*  applies an *ICellFunctor* on every cell of the grid --- if their corresponding amount of pheromon is greater than zero --- that draws a red mark on the screen according to their current state.

![Fig. 4.](http://www.plantuml.com/plantuml/svg/ZP91Yzim48Nl_XKF2iNU4hU7NffcJSEsMmAxj53ejKoiYIrG92EDaotB_lVAZjnWqqCF0V5cvdjvu_aKXBpqHvErJ8fzjZauAwYTSVvsRtgkxdLJudsvUJiKAlpKV6R_s7Ze0CAHXN0QDKXB0ceyDoGSS7IU1yt2MKuzPROJdBKns3Fwm0hYG4hXB-JWFBgMlJiwY_nxUbrS2rX-4eYBs8aOXedCQCi1-LUlrTHALi7jOxpQ3ALlwJcLprfQrmlgbcoZRJCYFHiIxMneTOSzovdPOjjr8smR2PvgkH0oZbBQMQZ9CDwL7p9jXVs1QiRkZesvxsqjpcEpONkgAZnd0F069-sb9uEJqmgkOq-nAw-ZiU3koyD3aRrabEnG6mfXuHw9AVwLUg7fRItwHC9vCPC_C_sxi0lZn5B-mHA3v_5tim3fq1dwdHSVH_aLdgT_-Z9rU3hdHQhkdJFuDNepRd4W4pYckuIQVzvPA7uIjcgPEsLTGKMFQJgForslPmj_NKmUruFz3_nobIfTKpwrFPAbXltDWB-2tMfLBSzZ4Qqig_b9hE5x6r1bhLQyNCH1ir53PNP7CtroJ-8V)

**PheromonViewer.h**
```c++
#ifndef PHEROMONVIEWER_H
#define PHEROMONVIEWER_H

#include "AbstractViewer.h"

namespace env
{
    class ICellFunctor;
};

namespace viewers
{
    class PheromonViewer : public AbstractViewer
    {
    public:
        PheromonViewer() = default;
        virtual ~PheromonViewer() = default;

    protected:
        virtual void iDraw();

    private:
        void drawPheromon(env::ICellFunctor &);
    };
} // namespace viewers
#endif // !PHEROMONVIEWER_H
```

## Demo

We are ready to instanciate our brand new *App* object with all the improvements for pheromon manipulation.

**main.cpp**

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
    env::Grid g;
    g.setSizeX(App::DEFAULT_WIDTH);
    g.setSizeY(App::DEFAULT_HEIGHT);
    g.setResolutionX(App::DEFAULT_RESX);
    g.setResolutionY(App::DEFAULT_RESY);
    g.iInitialize();
    app.setGrid(&g);

    //-- viewer
    viewers::GridViewer gviewer;
    app.setViewer(&gviewer);
    gviewer.initialize();
    gviewer.iActivate();

    // grid obstacles
    viewers::ObstacleViewer oviewer;
    oviewer.iActivate();

    // pheromons 
    viewers::PheromonViewer pviewer;
    pviewer.iActivate();

    // aggregator
    viewers::ViewerMgr mgr;
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

The interested reader can fork the complete source code from [here][5] and run the following in a terminal at the project folder root :

{% highlight shell %}
  # on windows
  $ cmake  -G "Visual Studio $(Version)" -S . -B ./build 
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


[1]: /posts/2d-grid-obstacles/
[2]: https://fr.wikipedia.org/wiki/Stigmergie
[3]: http://homepages.ulb.ac.be/~jldeneub/images/Deneubourgetal1990.pdf
[4]: https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Clock.php
[5]: https://github.com/kanmeugne/sfml2dgrid/releases/tag/sfml-2d-obstacles-pheromons
[6]: /posts/sfml-2d-grid/
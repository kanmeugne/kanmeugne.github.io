---
layout: page
sidebar: right
subheadline: Development
sidebar: right
title:  "Building a portable C++ SFML app with CMake"
teaser: "Building a portable <a href='https://www.sfml-dev.org/documentation/2.5.1/'> SFML </a> application can be a huge pain, disregarding the OS your working on - especially if you don't want to carry over binary files inside your source code repositories. This post presents a straightforward and simple way to package a project with external dependencies using CMake."
tags:
    - sfml cmake c++
categories:
    - dev
image:
    thumb: SFMLCMAKE.png
    title: SFMLCMAKE.png
    caption_url: http://unsplash.com
header:
   image_fullwidth: "header_homepage_13.jpg"
---

## Context

Few months ago, I found an [article][3] that explains how to build [GoogleTest and GoogleMock][4] directly in a [CMake][5] project. Because the approach is very straightforward, I managed to test it with [SFML][6] on a simple graphical app.

## Presentation of the Approach

In his article, [Craig Scott][3] explained how to build *GoogleTest* and *GoogleMock* directly in a *CMake project* and the configuration can be used almost as-is to add *SFML* dependencies as an *external project*. The main idea is to invoke *CMake* `ExternalProject` *command* and perform the build at *configure time*. This fully integrates the *external project* to your build and gives you access to all the *targets*. The author uses two sets of rules to configure the *build* :

### 1. CMakeLists.txt.in : external project references

All you need to know is the location of the official git repository of your external project and you are all set dfor this part.
You might need to use a specific tag though. 

{% highlight cmake %}
cmake_minimum_required(VERSION 3.8)
project(googletest-download NONE)

include(ExternalProject)
ExternalProject_Add(googletest
 GIT_REPOSITORY https://github.com/google/googletest.git
 GIT_TAG master
 SOURCE_DIR "${CMAKE_BINARY_DIR}/googletest-src"
 BINARY_DIR "${CMAKE_BINARY_DIR}/googletest-build"
 CONFIGURE_COMMAND ""
 BUILD_COMMAND ""
 INSTALL_COMMAND ""
 TEST_COMMAND ""
)
{% endhighlight %}

### 2. CMakeLists.txt : the configration file

This is where you define the targets. Note that the external project build is triggered before the target definitions.

{% highlight cmake %}
# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(
  COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
  WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download"
)
execute_process(
  COMMAND "${CMAKE_COMMAND}" --build .
  WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download"
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
add_subdirectory(
  "${CMAKE_BINARY_DIR}/googletest-src"
  "${CMAKE_BINARY_DIR}/googletest-build"
)
if(CMAKE_VERSION VERSION_LESS 2.8.11)
 include_directories(
   "${gtest_SOURCE_DIR}/include"
   "${gmock_SOURCE_DIR}/include"
 )
endif()

# Now simply link your own targets against gtest, gmock
{% endhighlight %}

With *CMakeLists.txt.in* and *CMakeLists.txt*, all we need to do is to call *CMake configuration* and *build* commands at the root of the project folder.

## Adaptation to SFML : Hello World app

We will implement this approach and build a simple *Hello World App* that display a window. Our project folder will match the following tree :

{% highlight shell %}
project/
├── App
│   ├── CMakeLists.txt
│   ├── CMakeLists.txt.in
│   ├── include
│   │   └── App.h
│   └── src
│       ├── App.cpp
│       └── main.cpp
├── CMakeLists.txt
├── bin
└── build
{% endhighlight %}

### App/src/main.cpp

The main file will basicaly launch two threads. One for the logic of the application - the main thread - and the other for the display routines. The SFML window should be initialized in the main thread.

{% highlight c++ %}
#include "App.h"
#include <thread>
#include <SFML/Graphics.hpp>

#ifdef __linux__
#include <X11/Xlib.h>
#endif

int main(){
#ifdef __linux__
  XInitThreads();
#endif
  sf::ContextSettings settings;
  settings.antialiasingLevel = 10;

  const unsigned int width = (App::DEFAULT_WIDTH*App::DEFAULT_RESX);
  const unsigned int height = (App::DEFAULT_HEIGHT*App::DEFAULT_RESY);

  sf::RenderWindow window (
    sf::VideoMode(width, height),
    "SFML & CMAKE",
    sf::Style::Titlebar | sf::Style::Close,
    settings
  );
  window.clear(sf::Color::Cyan);
  window.setFramerateLimit(120);
  window.setActive(false);

  App app;
  app.setWindow(&window);
  std::thread rendering_thread(&App::display, &app);
  app.run();
  rendering_thread.join();

  return 0;
}
{% endhighlight %}

### App/include/App.h

Logic and display functions are defined inside an *App* object.

{% highlight c++ %}
#ifndef APP_H
#define APP_H
namespace sf
{
 class RenderWindow;
};
class App
{
private:
 sf::RenderWindow* _window = nullptr;
public:
 static const unsigned int DEFAULT_WIDTH;
 static const unsigned int DEFAULT_HEIGHT;
 static const unsigned int DEFAULT_RESX;
 static const unsigned int DEFAULT_RESY;
 App(){};
 virtual ~App();
 void setWindow(sf::RenderWindow*);
 void run();
 void display();
};
#endif // !APP_H
{% endhighlight %}

### App/src/App.cpp

{% highlight c++ %}
#include "App.h"
#include <SFML/Graphics.hpp>
const unsigned int App::DEFAULT_WIDTH = 400;
const unsigned int App::DEFAULT_HEIGHT = 300;
const unsigned int App::DEFAULT_RESX = 2;
const unsigned int App::DEFAULT_RESY = 2;

void App::setWindow(sf::RenderWindow *w){
  _window = w;
}
App::~App(){
  _window = nullptr;
}
void App::display(){
  if (_window != nullptr){
    _window->setActive(true);
    while (_window->isOpen())
      _window->display();
  }
}
void App::run(){
  _window->setActive(false);
  while (_window->isOpen())
  {
    sf::Event event;
    while (_window->pollEvent(event)){
      if (event.type == sf::Event::Closed)
        _window->close();
      if (event.type == sf::Event::MouseButtonPressed){
        if (event.mouseButton.button == sf::Mouse::Right){
          int posx = event.mouseButton.x;
          int posy = event.mouseButton.y;
          printf("right mouse clicked.\nx=%d, y=%d\n", posx, posy);
        }
        if (event.mouseButton.button == sf::Mouse::Left){
          int posx = event.mouseButton.x;
          int posy = event.mouseButton.y;
          printf("left mouse clicked.\nx=%d, y=%d\n", posx, posy);
        }
      }
      if (event.type == sf::Event::MouseMoved){}
    }
  }
}
{% endhighlight %}

### App/CMakeLists.txt.in

Here, the *CMakeLists.txt.in* will hold references to the SFML official library.

{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)
project(sfml-download NONE)
include(ExternalProject)
ExternalProject_Add(sfml
  GIT_REPOSITORY    https://github.com/SFML/SFML.git
  GIT_TAG           master
  SOURCE_DIR        "${CMAKE_SOURCE_DIR}/build/sfml-src"
  BINARY_DIR        "${CMAKE_SOURCE_DIR}/build/sfml-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
{% endhighlight %}

### App/CMakeLists.txt


{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)

project (app C CXX)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt.in
  ${CMAKE_SOURCE_DIR}/build/sfml-download/CMakeLists.txt
)

execute_process(
  COMMAND ${CMAKE_COMMAND} -G ${CMAKE_GENERATOR} .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build/sfml-download
)

if(result)
  message(FATAL_ERROR "CMake step for sfml failed: ${result}")
endif()

execute_process(
  COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build/sfml-download
)

if(result)
  message(FATAL_ERROR "Build step for sfml failed: ${result}")
endif()

add_subdirectory(
  ${CMAKE_SOURCE_DIR}/build/sfml-src
  ${CMAKE_SOURCE_DIR}/build/sfml-build
)

set(SFML_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/build/sfml-build/include/)
set(APP_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include/)
file(GLOB_RECURSE APP_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
include_directories(${SFML_INCLUDE_DIR} ${APP_INCLUDE_DIR})

add_executable (app ${APP_SRC_FILES})

if(WIN32 OR WIN64)
  target_link_libraries(app sfml-window sfml-system sfml-graphics)
else()
  target_link_libraries(app sfml-window sfml-system
  sfml-graphics pthread X11)
endif()

source_group("src" FILES ${APP_SRC_FILES})
source_group("include" FILES ${APP_INCLUDE_DIR}/*.h)
{% endhighlight %}

We are almost set ! All we have to do now is run the *configuration* and the *build* processes.

We will need one more *CMakeLists.txt* file (the global one) that will be at the root of the project folder.

### CMakeLists.txt

{% highlight cmake %}
# CMakeList.txt : Upper level configuration file
cmake_minimum_required (VERSION 3.10)
set(
  CMAKE_RUNTIME_OUTPUT_DIRECTORY
  ${CMAKE_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}/
)
set(
  CMAKE_LIBRARY_OUTPUT_DIRECTORY
  ${CMAKE_SOURCE_DIR}/lib/${CMAKE_BUILD_TYPE}/
)
set(
  CMAKE_ARCHIVE_OUTPUT_DIRECTORY
  ${CMAKE_SOURCE_DIR}/build/${CMAKE_GENERATOR_PLATFORM}/${CMAKE_BUILD_TYPE}/
)
project (SFMLCMAKE C CXX)
add_subdirectory ("App")
{% endhighlight %}

**Note** : You can fork everything from [here][1]

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

## More like this
{: .t60 }
{% include list-posts tag='sfml cmake c++' %}

[1]: https://github.com/kanmeugne/sfmlcmake
[2]: https://www.sfml-dev.org/tutorials/2.5/compile-with-cmake.php
[3]: https://crascit.com/2015/07/25/cmake-gtest/
[4]: https://github.com/google/googletest
[5]: https://cmake.org/
[6]: https://www.sfml-dev.org/documentation/2.5.1/
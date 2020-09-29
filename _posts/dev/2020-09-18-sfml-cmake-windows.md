---
layout: page-fullwidth
sidebar: right
subheadline: Development
title:  "Building a portable C++ SFML app with CMake"
teaser: "Building a portable <a href='https://www.sfml-dev.org/documentation/2.5.1/'> SFML </a> application can be a huge pain, disregarding the OS your working on - especially if you don't want to carry over binary files inside your source code packages. This article present a simple and portable way to integrate all the dependencies."
breadcrumb: true
tags:
    - sfml cmake c++
categories:
    - dev
image:
    thumb: gallery-example-3-thumb.jpg
    title: gallery-example-3.jpg
    caption_url: http://unsplash.com
---

## Context

Few months ago, I found an [article](https://crascit.com/2015/07/25/cmake-gtest/ "Building GoogleTest and GoogleMock directly in a CMake project") that explains how to build [GoogleTest and GoogleMock](https://github.com/google/googletest "Welcome to Google Test, Google's C++ test framework!") directly in a [CMake](https://cmake.org/ "CMAKE Official Website") project. The approach is very straightforward, so I managed to test it with [SFML](https://www.sfml-dev.org/documentation/2.5.1/) on a simple graphical app.

## The Approach

[Craig Scott](https://crascit.com/2015/07/25/cmake-gtest/ "Building GoogleTest and GoogleMock directly in a CMake project") explained how to build GoogleTest and GoogleMock directly in a CMake project and the configuration can be used almost as-is to add SFML dependencies. The main idea is to invoke `ExternalProject` an perform the build at configure time. This fully integrate the external project to your build and gives you access to all the targets. 

In the [original article](https://crascit.com/2015/07/25/cmake-gtest/ "Building GoogleTest and GoogleMock directly in a CMake project"), the author uses two sets of rules to configure the build :

### Invoking the external project

The approach uses a `CMakeList.txt.in` to invoke the external project.

{% highlight cmake %}
cmake_minimum_required(VERSION 2.8.2)
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

### The configration file used to build the targets

The `CMakeList.txt` to be used as the sub project `CMakeList.txt`.

{% highlight cmake %}
# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
 WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download"
)
execute_process(COMMAND "${CMAKE_COMMAND}" --build .
 WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download"
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
add_subdirectory("${CMAKE_BINARY_DIR}/googletest-src"
 "${CMAKE_BINARY_DIR}/googletest-build"
)
if(CMAKE_VERSION VERSION_LESS 2.8.11)
 include_directories("${gtest_SOURCE_DIR}/include"
  "${gmock_SOURCE_DIR}/include"
 )
endif()

# Now simply link your own targets against gtest, gmock,
{% endhighlight %}

## Adaptation to SFML : A simple hello world app

To adapt this approach and build a simple *Hello World App* that display a window. Our project will match the following tree:


we will use an object called `App` and call it in a `main` function defined as follows : 

### main.cpp

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

The `App` Object is defined by the following *header file* and *source file* :

### App.h

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

### App.cpp

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

Before we move to the CMake configuration files, two remarks about the source code :
- Our application will be using two threads : one for the logic and a rendering thread
- Our window is designed as a 2D-grid (see you in futur posts !)
Now, the final touch ! We will use the following CMake files mimicking the approach use for *GoogleTest*

### CMakeList.txt.in

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

### CMakeList.txt

{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)
project (app C CXX)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt.in
 ${CMAKE_SOURCE_DIR}/build/sfml-download/CMakeLists.txt)
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

We are all set ! All we have to do now is run the configuration and the make process. We will need one more CMakeLists file (the global one) that will be at the root of the project.

{% highlight cmake %}
# CMakeList.txt : Upper level configuration file
cmake_minimum_required (VERSION 3.10)
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin/${CONFIG}/ )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib/${CONFIG}/ )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/build/${CMAKE_GENERATOR_PLATFORM}/${CONFIG}/)
project (SFMLCMAKE C CXX)
# sub projects
add_subdirectory ("App")
{% endhighlight %}

You can download the full code [here][1] and type the following in a terminal.

    > cmake ./build -G "Visual Studio 15 2017"

Then, in the `build` folder,

    > cmake --build --config Debug --target app

## Other Post Formats
{: .t60 }
{% include list-posts tag='post format' %}

 [1]: https://github.com/kanmeugne/sfmlcmake

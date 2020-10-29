---
layout: page-fullwidth
title:  "Portable C++ SFML app with CMake"
teaser: "Building a portable <a href='https://www.sfml-dev.org/documentation/2.5.1/'> SFML </a> application can be a huge pain, disregarding the OS you are working on (especially if you don't want to carry over binary files inside your source code repositories). In this post, I wanted to share an approach, that I have found very straigthforward, for packaging a project with external dependencies using CMake."
tags:
    - sfml
    - cmake
    - c++
    - modeling
categories:
    - modeling & simulation
image:
    thumb: SFMLCMAKE-thumb.png
---

Few months ago, I found an [article][3] that explains how to build [GoogleTest and GoogleMock][4] directly in a [CMake][5] project. Since the approach is amazingly straightforward, I have managed to test it with [SFML][6] on a simple graphical app.

The configuration used in the article can be exploited almost as-is to add *SFML* dependencies as an *external project* in a CMake-packaged app. The main idea is to invoke CMake *ExternalProject* command and perform the build at *configure time*. This fully integrates the *external project* to the build and gives access to all the *targets*. 

### The Hack

In the original post, the author used two sets of definitions to achieve this marvelous hack with googletest as the external project. First, a *CMakeLists.txt.in* file, that holds the external project references -- all you need to know is the location of the official git repository (or whatever compatible location) of your external project and you are all set for this part. 

**CMakeList.txt.in**
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

Second, you need to define your *CMakeLists.txt* for the targets of your application or your library. This is where the magic occurs! The external file is invoked at the beginning of the file, before the target definitions, as you can see in its content below. So when the targets are defined, it is possible to link with whatever dependencies we need.

**CMakeList.txt**
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

This is it! All you need to do is to call the call the proper cmake commands to configure and build your project. Now, this is how I copy the approach to build a little SFML Graphic App.

### Adaptation to build an SFML app

First, here is the project folder I have been using for the test :

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


The main file *App/src/main.cpp*  basically launches two threads : one for the logic of the application --- the main thread --- and the other, for the display routines (the SFML window should be initialized in the main thread). *App/include/App.h* and *App/src/App.cpp* defines the *App* object that will hold the logic and display functions.

To mimic the same approach, here is our SFML *CMakelists.txt.in*.

**App/CMakelists.txt.in**
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

*App/CMakeLists.txt* defines the target of the application --- namely the **app** executable --- set the sfml library dependencies to be built at configuration time.

**App/CMakeLists.txt**
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
# app target
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

The global *CMakeLists.txt* at the root folder is supposed to bundle everything up.

**./CMakeLists.txt**
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

### Configuration and build

If you have followed the steps above (and downloaded the source code available [here][1]), you should be able to type the following lines in a terminal at the root of the project folder --- for linux user, you might need to check [this][2] first.

{% highlight shell %}
  # on windows
  $ cmake  -G "Visual Studio 15 2017" -S . -B ./build 
  $ cmake  --build ./build --config Debug --target app

  # on linux
  $ mkdir build  
  $ cd build
  $ cmake -G "Unix Makefiles" .. -DCMAKE_BUILD_TYPE=Debug
  $ cmake --build ./ --target app --config Debug 
{% endhighlight %}

You should be able to launch the executable located in the bin folder with the command below and see a nice cyan window.

{% highlight shell %}
  bin> ./Debug/app
{% endhighlight %}

![screenshot](/images/sfmlcmake.jpg)

Smile! Now you are ready to take your graphical app everywhere you want. Enjoy and feel free to send me your feedbacks!

## More like this
{: .t60 }
{% include list-posts tag='sfml' %}

[1]: https://github.com/kanmeugne/sfmlcmake
[2]: https://www.sfml-dev.org/tutorials/2.5/compile-with-cmake.php
[3]: https://crascit.com/2015/07/25/cmake-gtest/
[4]: https://github.com/google/googletest
[5]: https://cmake.org/
[6]: https://www.sfml-dev.org/documentation/2.5.1/
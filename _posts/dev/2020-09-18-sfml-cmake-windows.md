---
layout: page
sidebar: right
subheadline: Development
title:  "Building a portable C++ SFML app with CMake"
teaser: "Building a portable <a href='https://www.sfml-dev.org/documentation/2.5.1/'> SFML </a> application can be a huge disregarding the OS your working on - especially if you don't want to carry over binaries files inside your source code packages. This article present a simple and portable way to integrate all the dependencies."
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

Few months ago, I found this [article](https://crascit.com/2015/07/25/cmake-gtest/ "Building GoogleTest and GoogleMock directly in a CMake project"), that explains how to build [GoogleTest and GoogleMock](https://github.com/google/googletest "Welcome to Google Test, Google's C++ test framework!") directly in a [CMake](https://cmake.org/ "CMAKE Official Website") project.

The approach is very straightforward, so I managed to test it with [SFML](https://www.sfml-dev.org/documentation/2.5.1/) on a simple graphical app.


While being very easy to use, packaging an SFML application can be tricky, especially if you want to share your source code to friends or clients, with different platforms, etc.

## The Approach

[Craig Scott](https://crascit.com/2015/07/25/cmake-gtest/ "Building GoogleTest and GoogleMock directly in a CMake project") explained how to build GoogleTest and GoogleMock directly in a CMake project and the configuration can be used almost as-is to add SFML dependencies.

The main idea is to invoke `ExternalProject` an perform the build at configure time. This fully integrate the external project to your build and gives you access to all the targets. 

In the [original article](https://crascit.com/2015/07/25/cmake-gtest/ "Building GoogleTest and GoogleMock directly in a CMake project"), the author uses two sets of rules to configure the build.

- `CMakeList.txt.in` to invoke the external project :

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

- `CMakeList.txt` to be used as the main project *CMakeLists.txt*:


{% highlight cmake %}
# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download"
)
execute_process(COMMAND "${CMAKE_COMMAND}" --build .
    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download"
)
# Prevent GoogleTest from overriding our compiler/linker options
# when building with Visual Studio
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
# Add googletest directly to our build. This adds the following targets:
# gtest, gtest_main, gmock and gmock_main
add_subdirectory("${CMAKE_BINARY_DIR}/googletest-src"
                 "${CMAKE_BINARY_DIR}/googletest-build"
)
# The gtest/gmock targets carry header search path dependencies
# automatically when using CMake 2.8.11 or later. Otherwise we
# have to add them here ourselves.
if(CMAKE_VERSION VERSION_LESS 2.8.11)
    include_directories("${gtest_SOURCE_DIR}/include"
                        "${gmock_SOURCE_DIR}/include"
    )
endif()

# Now simply link your own targets against gtest, gmock,
{% endhighlight %}

## Adaptation to SFML

We can use the same approach for SFML with the files below :

- `CMakeList.txt.in` to invoke [SFML project](https://github.com/SFML/SFML) (may by you chould consider using a different tag):

{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)

project(sfml-download NONE)

include(ExternalProject)
ExternalProject_Add(sfml
  GIT_REPOSITORY    https://github.com/SFML/SFML.git
  GIT_TAG           master
  SOURCE_DIR        "${CMAKE_CURRENT_SOURCE_DIR}/sfml-src"
  BINARY_DIR        "${CMAKE_CURRENT_SOURCE_DIR}/sfml-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
{% endhighlight %}

- `CMakeList.txt` to be used as the main project *CMakeLists.txt*:

{% highlight cmake %}
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/sfml.conf/CMakeLists.txt.in
    ${CMAKE_CURRENT_SOURCE_DIR}/sfml-download/CMakeLists.txt
)
execute_process(
	COMMAND ${CMAKE_COMMAND} -G ${CMAKE_GENERATOR} .
	RESULT_VARIABLE result
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/sfml-download
)
if(result)
	message(FATAL_ERROR "CMake step for sfml failed: ${result}")
endif()
execute_process(
	COMMAND ${CMAKE_COMMAND} --build .
	RESULT_VARIABLE result
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/sfml-download
)
if(result)
  message(FATAL_ERROR "Build step for sfml failed: ${result}")
endif()
# Add sfml directly to our build.
add_subdirectory(
	${CMAKE_CURRENT_SOURCE_DIR}/sfml-src
	${CMAKE_CURRENT_SOURCE_DIR}/sfml-build
)
# organize include and lib folders
set(SFML_INCLUDE_DIR
    ${CMAKE_CURRENT_SOURCE_DIR}/sfml-build/include/
)
set(SFML_LIBS
    ${CMAKE_CURRENT_SOURCE_DIR}/sfml-build/lib/${CMAKE_BUILD_TYPE}
)
include_directories(${SFML_INCLUDE_DIR})
link_directories(${SFML_LIBS})
# Now simply link your own targets against sfml libs
{% endhighlight %}

Note that I have introduced more global CMake variable for a better management :
- `${CMAKE_CURRENT_SOURCE_DIR}` is the current source directory. This should normally be you working directory
- `${CMAKE_BUILD_TYPE}` is the build type : *release*, *debug*, etc. It might be better to separated binaries depending on the build type (but it is personal).

Now let's deploy all this with the *sfml hello project* to see the magic happen.

## A simple project : SFML Hello World


## Other Post Formats
{: .t60 }
{% include list-posts tag='post format' %}

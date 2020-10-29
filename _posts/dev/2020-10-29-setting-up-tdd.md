---
layout: page-fullwidth
sidebar: right
title:  "Setting Up TDD with googletest (1/2)"
teaser: "Installs and validation of the test zero"
categories:
    - software development
tags:
    - cmake
    - c++
    - tdd
image:
    thumb: SFMLCMAKE-thumb.png
---

Le TDD fait référence à une approche de developpement informatique dans laquelle le code est toujours produit dans le but de valider des tests préalablement définis. Le but de cette approche est de garantir une qualité optimale du code à n'importe quelle étape du développement.

Le développeur qui adopte le TDD suit nécessairement [le cycle suivant][9] :
1. écrire des petits tests
2. s'assurer que les tests échouent dans un premier temps
3. écrire le code qui permet passer le test
4. s'assurer que le test passe
5. nettoyer ou réfactorer le code
6. s'assurer que le test passe toujours
7. retourner à l'étape 1.

Le TDD est une approche rigoureuse qui peut être coûteuse à mettre en oeuvre, mais qui apporte beaucoup de sérénité pour les developpeurs sur le long terme.

Cet article propose un petit tutoriel pour s'initier à l'approche TDD. On fera semblant de développer une bibliothèque C++ en utilisant les technologies suivantes :
- [CMake][1] pour le packaging (à installer)
- [googletest][4] pour la gestion des tests unitaires
- [Git][2] pour l'intégration continue et la gestion de version

### The example : libtoolset.a (or toolset.lib)

On se propose de developper une librairie de traitement de texte -- __libtoolset.a__ ou __toolset.lib__ *(sous windows)* -- qui contient un objet *parser* avec les fonctionnalités suivantes:
- *convertToLowerCase* : convertit un mot ou une phrase en minuscule
- *convertToUpperCase* : convertit un mot ou une phrase en majuscule

### Test zero : the code should compile

TDD oblige, on commence par mettre en place l'infastructure minimale qui permet de valider les tests de compilation. En d'autres termes, on doit pouvoir lancer les commandes suivantes :

{% highlight bash %}
    > cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..
    > cmake --build . --config Debug
    > ../bin/Debug/toolset_test
{% endhighlight %}

Cela suppose un makefile qui définit les bonnes cibles pour la lib -- __libtoolset.a__ -- et qui génère l'exécutable [googletest][4] pour les tests unitaires __toolset_test__ -- ou __toolset_test.exe__ *sous windows*. 

Pour cela, partons de l'arborescence suivante : 

{% highlight bash %}
project
├── CMakeLists.txt # makefile global
├── bin # stockage des executables
├── lib # stockage des librairies
├── build # fichiers de builds
├── deps # definitions des dépendances externes
├── tests
│   ├── CMakeLists.txt # makefile pour les tests
│   ├── include
│   └── src
└── toolset
    ├── CMakeLists.txt # makefile pour la lib
    ├── include
    └── src
{% endhighlight %}

Les sources pour définir __libtoolset.a__ et __toolset_test__ seront stockées respectivement dans le dossier __toolset/__ et __tests/__. Les définitions globales du projet sont contenues dans le fichier __./CMakeLists.txt__:

{% highlight cmake %}
# CMakeList.txt : Upper level configuration file
cmake_minimum_required (VERSION 3.8)

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
    ${CMAKE_SOURCE_DIR}/lib/${CMAKE_BUILD_TYPE}/
)

project (toolset C CXX)

# sub projects
add_subdirectory ("toolset")
add_subdirectory("tests")
{% endhighlight %}

**Note**: les premières lignes de __./CMakeLists.txt__ définissent les chemins par défaut pour les cibles et les fichier intermédianres, selon les configurations (Release ou Debug), et suivant les architectures. Les deux dernières lignes indiquent que le projet contient deux sous-projets : un pour les **tests** et l'autre pour la lib **toolset**.

Bien évidemment dans l'état actuel, la compilation ne fonctionne pas car les sous-projets sont vides, l'exécution des commandes ci dessus renvoient un code d'erreur.

Pour définir les sous projets, commençons par remplir le fichier __./toolset/CMakeLists.txt__ pour la cible __libtoolset.a__:

{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)

set(BINARY ${CMAKE_PROJECT_NAME})

######################################
# organize include and src files

set(TOOLSET_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include/)
set(TOOLSET_INCLUDE_DIR ${TOOLSET_INCLUDE_DIR} PARENT_SCOPE)
file(GLOB_RECURSE TOOLSET_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
include_directories(${TOOLSET_INCLUDE_DIR})

add_library(${BINARY} ${TOOLSET_SRC_FILES})
{% endhighlight %}

puis le fichier __./tests/CMakeLists.txt__ pour l'exécutable de test __toolset_test__:

{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)

set(BINARY ${CMAKE_PROJECT_NAME}_test)

#################################
# Configure and build GoogleTest
configure_file(
    ${CMAKE_SOURCE_DIR}/deps/gtest/CMakeLists.txt.in
    ${CMAKE_SOURCE_DIR}/build/googletest-download/CMakeLists.txt
)
execute_process(
	COMMAND ${CMAKE_COMMAND} -G ${CMAKE_GENERATOR} .
	RESULT_VARIABLE result
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build/googletest-download
)
if(result)
	message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(
	COMMAND ${CMAKE_COMMAND} --build .
	RESULT_VARIABLE result
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build/googletest-download
)
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()
# Prevent overriding the parent project's compiler/linker
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
add_subdirectory(
	${CMAKE_SOURCE_DIR}/build/googletest-src
	${CMAKE_SOURCE_DIR}/build/googletest-build
	EXCLUDE_FROM_ALL
)
#################################
# organize include and src files
set(
	GTEST_INCLUDE_DIR
	${CMAKE_SOURCE_DIR}/build/googletest-src/googlemock/include/
	${CMAKE_SOURCE_DIR}/build/googletest-src/googletest/include/
)
set(
	TEST_INCLUDE_DIR
	${CMAKE_CURRENT_SOURCE_DIR}/include/
)
file(
	GLOB_RECURSE
	TEST_SRC_FILES
	${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
)
include_directories(
	${GTEST_INCLUDE_DIR}
	${TEST_INCLUDE_DIR}
	${TOOLSET_INCLUDE_DIR}
)
add_executable (${BINARY} ${TEST_SRC_FILES})
target_link_libraries(${BINARY} ${CMAKE_PROJECT_NAME} gmock_main)
{% endhighlight %}

Le fichier de définition pour la cible __libtoolset.a__ est facile à comprendre. On indique que les entêtes se trouvent dans le  dossier __./toolset/include/__ et les sources, dans le dossier .__/toolset/src/__. On précise également le nom de la cible à la fin du fichier en utilisant la varibale __${CMAKE_PROJECT_NAME}__ définie dans le __CMakeLists.txt__ global.

Le fichier __./tests/CMakeLists.txt__ est un peu plus complexe. On se sert ici d'une astuce proposée dans cet [article][6] qui consiste à définir __googletest__ comme une dépendance extérieure et générer les cibles au moment de la configuration. La dépendance à googletest est déclarée dans un fichier de configuration intermédiaire --- __./deps/gtest/CMakeLists.txt.in__ (remarque : on peut utiliser le même procédé pour intégrer d'autres dépendances externes) :

{% highlight cmake %}
cmake_minimum_required (VERSION 3.8)

project(googletest-download NONE)

include(ExternalProject)
ExternalProject_Add(googletest
  GIT_REPOSITORY    https://github.com/google/googletest
  GIT_TAG           release-1.10.0
  SOURCE_DIR        "${CMAKE_SOURCE_DIR}/build/googletest-src"
  BINARY_DIR        "${CMAKE_SOURCE_DIR}/build/googletest-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
{% endhighlight %}

**Note**: le fichier __./deps/gtest/CMakeLists.txt.in__ est utilisé dans __./tests/CMakeLists.txt__ au moment de la configuration et de la création des cibles __googletest__.

 Le nom de la cible pour l'exécution des tests est définie à la dernière ligne du fichier __./tests/CMakeLists.txt__.

Dans l'état actuel, les commandes de notre **test zero** ne fonctionnent toujours pas. Notamment, la commande

{% highlight bash %}
> cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..
{% endhighlight %}

renvoie un code d'erreur car les fichiers sources pour les sous project __tests__ et __toolset__ sont inexistants! On complète l'initialisation du projet avec le header __./toolset/include/MyParser.h__ pour déclarer le *parser*, 

{% highlight c++ %}
#ifndef MYPARSER_H

class MyParser;

#endif // MYPARSER_H
{% endhighlight %}

le fichier source correspondant __./toolset/src/MyParser.cpp__,

{% highlight c++ %}
#include "MyParser.h"
{% endhighlight %}

puis le fichier source__./tests/src/main.cpp__ pour la définition et l'exécution des tests :

{% highlight c++ %}
#include <gmock/gmock.h>

using namespace testing;
int main(int argc, char** argv)
{
    testing::InitGoogleMock(&argc, argv);
    return RUN_ALL_TESTS();
}
{% endhighlight %}

Dans l'état actuel notre test zero passe! Les commandes :

{% highlight bash %}
> cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..
build> cmake --build . --config Debug
{% endhighlight %}

génère un makefile (fichier __.sln__ sous windows avec [visual studio][7]). A la fin de la compilation, on a les cibles correctement générée:

{% highlight bash %}
#(on linux - ubuntu)
.
├── bin
│   └── Debug
│       └── toolset_test
├── CMakeLists.txt
├── deps
│   └── gtest
│       └── CMakeLists.txt.in
├── lib
│   └── Debug
│       └── libtoolset.a
├── tests
│   ├── CMakeLists.txt
│   └── src
│       └── main.cpp
└── toolset
    ├── CMakeLists.txt
    ├── include
    │   └── MyParser.h
    └── src
        └── MyParser.cpp
{% endhighlight %}

La commande :

{% highlight bash %}
build> ../bin/Debug/toolset_test
{% endhighlight %}

s'exécute normalement et renvoie:

{% highlight bash %}
[==========] Running 0 tests from 0 test suites.
[==========] 0 tests from 0 test suites ran. (0 ms total)
[  PASSED  ] 0 tests.
{% endhighlight %}

Evidemment, aucun test unitaire n'a été défini pour l'instant. C'est maintenant dans le cercle vertueux du TDD.

**Note**: les sources sont disponibles sur [github][8], pour les impatients.



## Articles similaires

{% include list-posts tag='c++' %}


[1]: https://cmake.org/
[2]: https://git-scm.com/
[3]: https://github.com/
[4]: https://github.com/google/googletest
[5]: https://crascit.com/2015/07/25/cmake-gte
[6]: https://travis-ci.org/
[7]: https://docs.microsoft.com/fr-fr/cpp/build/cmake-projects-in-visual-studio?view=vs-2019
[8]: https://github.com/google/googletest
[9]: https://en.wikipedia.org/wiki/Test-driven_development
---
sidebar: right
title:  "Introduction au TDD (I) : Mise en place avec googletest"
teaser: "Installation de l'environnement de test et validation du test zero"
categories:
    - software development
tags:
    - cmake
    - c++
    - tdd
comments: true
author: kanmeugne
---


Le [TDD][9] --- **Test-Driven Development** --- fait référence à une approche de developpement informatique dans laquelle le code est toujours produit dans le but de valider des tests préalablement définis. Le but de cette approche est de garantir une qualité optimale du code à n'importe quelle étape du développement.

![Introduction au TDD Poster](/images/test11-unsplash.jpg){: width="500"}
_Photo by [Todd Quackenbush](https://unsplash.com/@toddquackenbush?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)_

## Le Mindset

Le développeur qui adopte le *TDD* suit nécessairement [le cycle suivant][9] :
1. Écrire des petits tests
2. S'assurer que les tests échouent dans un premier temps
3. Écrire le code qui permet passer le test
4. S'assurer que le test passe
5. Nettoyer ou réfactorer le code
6. S'assurer que le test passe toujours
7. Retourner à l'étape 1.

> Le [TDD][9] --- **Test-Driven Development** --- fait référence à une approche de developpement informatique dans laquelle le code est toujours produit dans le but de valider des tests préalablement définis. Le but de cette approche est de garantir une qualité optimale du code à n'importe quelle étape du développement.
{: .prompt-tip }

## Mise en situation : développement de la lib *toolset*

Le *TDD* est une approche rigoureuse qui peut être coûteuse à mettre en oeuvre, mais qui apporte beaucoup de sérénité pour les developpeurs sur le long terme. Dans ce post, je propose un petit tutoriel pour s'initier à l'approche *TDD*. On fera semblant de développer une bibliothèque C++ en utilisant les technologies suivantes :
- [CMake][1] pour le packaging (à installer)
- [googletest][4] pour la gestion des tests unitaires
- [Git][2] pour l'intégration continue et la gestion de version (à installer)

La bibliothèque contiendra un *parser* avec les fonctionnalités suivantes:
- *convertToLowerCase* : convertit un mot ou une phrase en minuscule
- *convertToUpperCase* : convertit un mot ou une phrase en majuscule
  
Le but de cet exercice est d'appréhender les bonnes pratiques du *TDD* et de comprendre leur intérêt.

## Le test *zero* : il faut que ça compile !

La première contrainte que l'on se fixe c'est de pouvoir lancer les commandes ci-dessous, car le test de compilation est le test le plus fondamental!

{% highlight bash %}
    > cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..
    > cmake --build . --config Debug
    > ../bin/Debug/toolset_test
{% endhighlight %}
> Le test de compilation est le test le plus fondamental !
{: .prompt-warning }

Cela suppose de définir les bonnes cibles pour la lib *toolset* et pour les *tests unitaires*.

## Organisation du workspace

Pour attaquer la production du code qui permettra de valider le test *zero*, partons de l'arborescence ci-dessous, qui organise le projet en deux sous projets : un pour les *tests* et un autre pour la lib *toolset*.

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
> Le projet s'organise en deux sous projets. Un pour la lib toolset et un autre pour les tests.
{: .prompt-tip }

Les sources pour définir la lib *toolset* et l'exécutable de *tests* sont stockées respectivement dans les dossiers `toolset/` et `tests/`. Le makefile global du projet est défini dans `./CMakeLists.txt`, à la racine du dossier.

### Makefile global du projet

```cmake
# CMakeList.txt : Upper level configuration file
cmake_minimum_required (VERSION 3.8)

# global paths
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
    ${CMAKE_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}/)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY
    ${CMAKE_SOURCE_DIR}/lib/${CMAKE_BUILD_TYPE}/)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY 
    ${CMAKE_SOURCE_DIR}/lib/${CMAKE_BUILD_TYPE}/)

# project declaration
project (toolset C CXX)
# sub projects
add_subdirectory ("toolset")
add_subdirectory("tests")
```
> Les lignes 4---10 de `./CMakeLists.txt` définissent les chemins par défaut pour les cibles et les fichiers intermédiaires, selon les configurations (Release ou Debug), et suivant les architectures. Les lignes 14---16 indiquent que le projet contient deux sous-projets : un pour les **tests** et l'autre pour la lib **toolset**.
{: .prompt-tip }

Bien évidemment dans l'état actuel, la compilation échoue puisque les dossiers des sous-projets sont vides. Pour les configurer, on commence par remplir le fichier `./toolset/CMakeLists.txt`.

### Makefile pour la cible *toolset*

```cmake
cmake_minimum_required (VERSION 3.8)
set(BINARY ${CMAKE_PROJECT_NAME})

################################
# organize include and src files
set(TOOLSET_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include/)
set(TOOLSET_INCLUDE_DIR ${TOOLSET_INCLUDE_DIR} PARENT_SCOPE)
file(GLOB_RECURSE TOOLSET_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
include_directories(${TOOLSET_INCLUDE_DIR})
add_library(${BINARY} ${TOOLSET_SRC_FILES})
```
> Le fichier indique que les entêtes se trouvent dans le  dossier `./toolset/include/` (ligne 9) et les sources, dans le dossier .`/toolset/src/` (lignes 8 et 10). Le nom de la cible est indiqué à la ligne 11 en utilisant la varibale `${CMAKE_PROJECT_NAME}`, définie dans le `CMakeLists.txt` global du projet.
{: .prompt-tip }

Après le makefile de la lib *toolset*, on remplit le fichier `./tests/CMakeLists.txt` pour les *tests unitaires*. Pour cela, on se sert d'une [astuce][5] qui consiste à définir *googletest* comme une dépendance extérieure et à générer toutes ses cibles au moment de la configuration --- la dépendance à *googletest* est déclarée dans un fichier de configuration intermédiaire, `./deps/gtest/CMakeLists.txt.in`.

### Makefile pour les tests

```cmake
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
    ${CMAKE_CURRENT_SOURCE_DIR}/include/)
file(
    GLOB_RECURSE
    ${TEST_SRC_FILES}
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
)
include_directories(
    ${GTEST_INCLUDE_DIR}
    ${TEST_INCLUDE_DIR}
    ${TOOLSET_INCLUDE_DIR}
)
add_executable (${BINARY} ${TEST_SRC_FILES})
target_link_libraries(${BINARY} ${CMAKE_PROJECT_NAME} gmock_main)
```
> le makefile des tests fait appel à une dépendance extérieure (ligne 5) pour générer les entêtes et les cibles de *googletest* à la configuration (lignes 5---30). La cible pour l'exécutable des tests est complètement définie de la ligne 51 --- 52.
{: .prompt-tip }

Le fichier `./tests/CMakeLists.txt.in` utilisé pour la dépendence à googletest indique le lien github officiel des sources et le tag à utiliser.

```cmake
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
```
> Le fichier `./deps/gtest/CMakeLists.txt.in` est utilisé dans `./tests/CMakeLists.txt` au moment de la configuration et de la création des cibles `googletest`. Il indique le lien github pour récupérer les sources et le tag à utiliser.
{: .prompt-tip }

### Code source *TDD*--compatible

Dans l'état actuel, la production de code ne valide toujours pas le test *zero* . Notamment, la commande :

{% highlight bash %}
> cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..
{% endhighlight %}

renvoie un code d'erreur car les fichiers sources pour les sous projets `tests` et `toolset` sont inexistants! On complète l'initialisation du projet avec le strict minimum pour pouvoir valider la compilation. 

Premièrement, le header `./toolset/include/MyParser.h` pour déclarer le *parser* :

```c++
#ifndef MYPARSER_H

class MyParser;

#endif // MYPARSER_H
```

Ensuite, le fichier source `./toolset/src/MyParser.cpp`, qui ne contient qu'une ligne pour l'instant :

```c++
#include "MyParser.h"
```

Enfin, le fichier source`./tests/src/main.cpp` pour la définition et l'exécution des tests :

```c++
#include <gmock/gmock.h>

using namespace testing;
int main(int argc, char** argv)
{
    testing::InitGoogleMock(&argc, argv);
    return RUN_ALL_TESTS();
}
```

### Validation du test zero

La production de code fournie ci-dessus valide notre test *zero* ! La commande `cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..` génère un makefile système (ou fichier `.sln` sous windows avec [visual studio][7]), et la commande `cmake --build . --config Debug` génère les bonnes cibles comme on peut le voir sur l'arborescence ci-dessous.

{% highlight bash %}
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
> Résultat de la commande `cmake --build . --config Debug` sur une machine ubuntu avec gcc-7.5. La lib *toolset* --- `./lib/Debug/libtoolset.a` --- est correctement générée ainsi que l'exécutable pour les *tests* --- `./bin/Debug/toolset`.
{: .prompt-tip }

Pour l'instant lorsqu'on lance l'exécutable de *tests* on obtient un message qui nous indique qu'aucun test n'a été défini --- ce qui est tout à fait normal. La suite de l'exercice consiste à définir les tests unitaires qui permettront de valider chacune des fonctionnalités de la lib toolset à terme. 

{% highlight bash %}
> ./bin/Debug/toolset

[==========] Running 0 tests from 0 test suites.
[==========] 0 tests from 0 test suites ran. (0 ms total)
[  PASSED  ] 0 tests.
{% endhighlight %}
> Les sources sont disponibles sur [github][8], pour les impatients.
{: .prompt-tip }

Maintenant que la mise en place de l'espace de travail est faite, on peut véritablement entrer dans le cycle vertueux des *TDD*.

## Conclusion

Le plus dur du travail est fait avec cette mise en place. C'est très important de valider le test *zero* car c'est la condition nécessaire pour travailler itérativement par la suite.

Dans la [deuxième partie][10] de cet exercice, on va se concentrer sur l'implémentation des fonctionnalités de la lib. On verra que l'approche TDD eprmet d'envisager sereinement l'évolution du code et le refactoring.

## Resources

- [https://cmake.org/cmake/help/v3.8/](https://cmake.org/cmake/help/v3.8/)
- [https://git-scm.com/](https://git-scm.com/)
- [https://github.com/google/googletest](https://github.com/google/googletest)
- [https://crascit.com/2015/07/25/cmake-gte](https://crascit.com/2015/07/25/cmake-gte)
- [https://docs.microsoft.com/fr-fr/cpp/build/cmake-projects-in-visual-studio?view=vs-2019](https://docs.microsoft.com/fr-fr/cpp/build/cmake-projects-in-visual-studio?view=vs-2019)
- [https://en.wikipedia.org/wiki/Test-driven_development](https://en.wikipedia.org/wiki/Test-driven_development)

[1]: https://cmake.org/cmake/help/v3.8/
[2]: https://git-scm.com/
[8]: https://github.com/kanmeugne
[4]: https://github.com/google/googletest
[5]: https://crascit.com/2015/07/25/cmake-gte
[7]: https://docs.microsoft.com/fr-fr/cpp/build/cmake-projects-in-visual-studio?view=vs-2019
[9]: https://en.wikipedia.org/wiki/Test-driven_development
---
title:  "Introduction au TDD (II) : le cycle vertueux"
teaser: "Entering the virtuous cycle"
categories:
    - software development
tags:
    - cmake
    - c++
    - tdd
comments: true
author: kanmeugne
---


Dans la [première partie][6] de cet exercice, je me suis concentré sur la mise en place de l'espace de travail et la validation du test zero (qui est tout simplement la compilation). On va se concentrer maintenant sur l'implémentation des fonctionnalités de la lib témoin. On verra en quoi l'approche TDD permet d'envisager sereinement l'évolution du code et le refactoring.

![Introduction au TDD Poster](/images/test12-unsplash.jpg){: width="500"}
_Photo by [Liam Tucker](https://unsplash.com/@itsliamtucker?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)_

## Le mindset

Pour rappel, le [developpeur TDD][6] respecte le cycle suivant :
1. Écrire des petits tests
2. S'assurer que les tests échouent dans un premier temps
3. Écrire le code qui permet passer le test
4. S'assurer que le test passe
5. Nettoyer (ou réfactorer) le code
6. S'assurer que le test passe toujours
7. Retourner à l'étape 1.

La particularité de cette approche est qu'on définit les tests en premier lieu --- la production de code a pour objectif de les valider.

## Le premier test

Pour commencer la production de notre lib *toolset*, il faut donc prélablement définir un test qui est sensé échouer dans l'état actuel du code. Commençons par le test ci-dessous : 

```c++
class ParserTest : public Test{};

TEST(ParserTest, Parser_LowerSingleLetter)
{
    std::string output = "";
    std::string input = "L";
    std::string expected = "l";
    MyParser parser;
    parser.convertToLowerCase(input, output);
	ASSERT_EQ(output, "l");
}
// ...
```

> **Traduction en langage naturel** : La fonction `convertToLowerCase` du parser convertit la lettre "L" majuscule en "l" minuscule
{: .prompt-tip }

Quelques mots clefs dans cette définition nécessitent une petite explication :
- `TEST` est la macro *googletest* qui permet de définir un test unitaire. `ASSERT_EQ` est une autre MACRO qui permet de tester si 2 variables ont la même valeur (voir la [documentation de googletest][4] pour plus d'infos sur les macros disponibles).
- La déclaration de la classe `ParserTest` (dérivée de `testing::Test`) permet de regrouper les tests par thématique --- comme on le verra plus bas, ce mécanisme permet aussi de définir des [fixtures][4].
- `Parser_LowerSingleLetter` : est le nom du test. Très utile quand il faudra lire les résultats des tests sur la console.

Le test défini ci-dessus est simple (une assertion) avec un objectif exprimable en langage naturel : *le parseur doit transformer la lettre L (majuscule) en la lettre l (minuscule)*. Produisons maintenant le code qui permet de le valider.

## La première validation

Pour l'instant le test échoue à cause d'un problème de compilation puisque la classe `MyParser` n'est pas vraiment définie. En écrivant le strict minimum dans `MyParser.h` pour que la compilation fonctionne,

```c++
#include "MyParser.h"

//MyParser::convertToLowerCase
void MyParser::convertToLowerCase(const std::string &input, std::string &output)
{
    output = "";
}
//MyParser::MyParser
MyParser::MyParser()
{
}
//MyParser::~MyParser
MyParser::~MyParser()
{
}
```

on obtient un message d'erreur plus conventionnel :

```bash
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from ParserTest
[ RUN      ] ParserTest.Parser_LowerSingleLetter
/.../tests/src/main.cpp:15: Failure
Expected equality of these values:
  output
    Which is: ""
  "l"
[  FAILED  ] ParserTest.Parser_LowerSingleLetter (0 ms)
[----------] 1 test from ParserTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 0 tests.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] ParserTest.Parser_LowerSingleLetter

 1 FAILED TEST
```

En d'autre termes, le test `Parser_LowerSingleLetter` échoue car la valeur obtenue par le *parser* --- la chaine de caractère vide --- ne correspond pas à la valeur attendue --- "l".

Pour que le test soit validé, il faut produire une implémentation correcte de la fonction `MyParser::convertToLowerCase` dans `MyParser.cpp`. Pour cela, disons que la fonction **parcourt la chaine de caractères, transforme en minuscules tous les caractères et les rajoute dans la variable de sortie**. Ce qui nous donne l'implémentation suivante :

```c++
//MyParser::convertToLowerCase
void MyParser::convertToLowerCase(const std::string &input, std::string &output)
{
    output = "";
    for (auto c : input)
        output.push_back(tolower(c));
}
```

Avec cette implémentation, le test est OK. On a produit le code qui valide notre premier test unitaire --- je laisse au lecteur le soin de rajouter d'autres tests sur cette première fonction pour éprouver le code s'il le souhaite.

```bash
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from ParserTest
[ RUN      ] ParserTest.Parser_LowerSingleLetter
[       OK ] ParserTest.Parser_LowerSingleLetter (0 ms)
[----------] 1 test from ParserTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test.
```
> La production de code valide le test, on peut avancer dans le développement.
{: .prompt-tip }

## Red, Green, Refactor

Ecrivons notre deuxième test pour la deuxième fonction :

```c++
TEST(ParserTest, Parser_UpperSingleLetter)
{
    std::string output = "";
    std::string input = "l";
    std::string expected = "L";
    MyParser parser;
    parser.convertToUpperCase(input, output);
	ASSERT_EQ(output, "L");
}
```

Comme le premier, le deuxième test échoue puisque la fonction `MyParser::convertToUpperCase` n'est pas définie. Effectuons les modifications de code qui permettent de valider le test.

Tout d'abord, la mise à jour des définitions dans le fichier `./toolset/include/MyParser.h`,

```c++
class MyParser
{
public:
    MyParser();
    ~MyParser();

    void convertToLowerCase(const std::string &, std::string &);
    void convertToUpperCase(const std::string &, std::string &);
};
```

puis l'implémentation de la fonction `convertToUpperCase` dans le fichier `./toolset/src/MyParser.cpp`,

```c++
//MyParser::convertToUpperCase
void MyParser::convertToUpperCase(const std::string &input, std::string &output)
{
    output = "";
    for (auto c : input)
        output.push_back(toupper(c));
}
```
L'exécution des tests renvoie maintenant le résultat suivant :

```bash
[==========] Running 2 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 2 tests from ParserTest
[ RUN      ] ParserTest.Parser_LowerSingleLetter
[       OK ] ParserTest.Parser_LowerSingleLetter (0 ms)
[ RUN      ] ParserTest.Parser_UpperSingleLetter
[       OK ] ParserTest.Parser_UpperSingleLetter (0 ms)
[----------] 2 tests from ParserTest (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test suite ran. (0 ms total)
[  PASSED  ] 2 tests.
```

On voit que le premier test reste valide --- ce qui signifie qu'il n'y a pas eu de regression --- et que le deuxième test est également OK. La logique de production est toujours la même et peut se résumer en une formule synthétique : **Red**, **Green**, **Refactor** :
- **Red** (le test échoue d'abord),
- **Green** (le test passe après une modification du code, tous les tests précédents doivent toujours être valides)
- **Refactor** (on restructure le code pour une cohérence d'ensemble et on revérifie que tous les tests passent toujours)
  
On remarque le cycle vertueux qui oblige à avancer lentement mais surement en faisant le moins de dégâts possible.


## Cas d'usage de Refactoring

L'approche *TDD* se prête également bien aux tâches de refactoring pures. Dans ces cas, on profite des tests déjà définis pour controler la qualité du code.

Pour notre exemple, essayons un refactoring en deux étapes :
1. Création d'un namespace `utils`, qui contiendra les définitions de la classe `MyParser`.
2. Ajout d'une fonction dans la bibliothèque qui renvoie une instance unique de `MyParser`

### Refactoring 1 : ajouter un namespace

Pour la première étape, modifions tout d'abord les tests qui doivent valider le code. La logique des tests n'est pas modifiée --- ce qui serait une faute grave --- juste l'initialisation du *parser*.

```c++
TEST(ParserTest, Parser_LowerSingleLetter)
{
    std::string output = "";
    std::string input = "L";
    std::string expected = "l";
    utils::MyParser parser; // namespace utils
    parser.convertToLowerCase(input, output);
	ASSERT_EQ(output, "l");
}

TEST(ParserTest, Parser_UpperSingleLetter)
{
    std::string output = "";
    std::string input = "l";
    std::string expected = "L";
    utils::MyParser parser; // namespace utils
    parser.convertToUpperCase(input, output);
	ASSERT_EQ(output, "L");
}
```

Naturellement --- et heureusement --- ces tests échouent dans un premier temps puisque `MyParser.h` n'est pas défini dans le bon namespace. Les modifications suivantes vont permettre de les valider.

D'abord le fichier `./toolset/include/MyParser.h`, 
```c++
#ifndef MYPARSER_H
#include <string>

namespace utils
{
    class MyParser
    {
    public:
        MyParser();
        ~MyParser();

        void convertToLowerCase(const std::string &, std::string &);
        void convertToUpperCase(const std::string &, std::string &);
    };
} // namespace utils
#endif // MYPARSER_H
```

Puis le fichier `./toolset/src/MyParser.cpp`, dans lequel il suffira de rajouter la ligne ci-dessous :

```c++
using namespace utils;
```

Les tests refonctionnent! Refactoring réussi en toute sérénité.

Maintenant voyons pour le deuxième refactoring.

### Refactoring 2: utiliser une référence unique

Pour la deuxième étape du refactoring, l'idée est de rajouter une fonction --- `utils::getParser` --- qui renvoie une instance unique de `MyParser`. On testera cette fonction en utilisant une nouvelle classe dans le fichier `./tests/main.cpp` :
```c++
//Les fixtures sont des attributs publiques des classes Test
class UniqueParserTest : public Test
{
public:
    utils::MyParser_t &_parser;

    UniqueParserTest():_parser(utils::getParser()) {}
};
```

La classe `UniqueParserTest` définit une fixture `UniqueParserTest::_parser` qui est initialisée dans `UniqueParserTest::UniqueParserTest()`.

Les tests `UniqueParserTest` vont permettre de valider, d'une part, que la fonction `utils::getParseur` retoune toujours la même instance, et d'autre part, que toutes les fonctionnalités du parseur sont bien validées par cette instance.

On définit les nouveaux tests suivants --- on notera l'utilisation de la macro `TEST_F` à la place de `TEST`, ce qui permet exploiter les fixtures dans le test :

```c++
TEST_F(UniqueParserTest, Parser_UniqueParserIsUnique)
{
    utils::MyParser_t& p = utils::getParser();
    ASSERT_EQ(std::addressof(p), std::addressof(_parser));
}

TEST_F(UniqueParserTest, Parser_UniqueParserLowerSingleLetter)
{
    std::string output = "";
    std::string input = "L";
    std::string expected = "l";
    _parser->convertToLowerCase(input, output);
    ASSERT_EQ(output, "l");
}
TEST_F(UniqueParserTest, Parser_UniqueParserUpperSingleLetter)
{
    std::string output = "";
    std::string input = "l";
    std::string expected = "L";
    _parser->convertToUpperCase(input, output);
    ASSERT_EQ(output, "L");
}
```
On notera que le test prévoit l'utilisation du type `MyParser_t` -- à définir dans le code à produire -- pour stocker l'instance unique de `MyParser`.

Bonne nouvelle, les tests échouent dans un premier temps, puis (après plusieurs essais), les modifications suivantes permettent de les valider.

Tout d'abord le fichier `MyParser.h` :

```c++
#ifndef MYPARSER_H
#include <string>
#include <memory>

namespace utils
{
    class MyParser
    {
    public:
        MyParser();
        ~MyParser();

        void convertToLowerCase(const std::string &, std::string &);
        void convertToUpperCase(const std::string &, std::string &);
    };

    typedef std::unique_ptr<MyParser> MyParser_t;

    MyParser_t& getParser();
} // namespace utils
#endif // MYPARSER_H
```

puis, le fichier `MyParser.cpp` :

```c++
#include "MyParser.h"
using namespace utils;

//MyParser::convertToLowerCase
void MyParser::convertToLowerCase(const std::string &input, std::string &output)
{
    output = "";
    for (auto c : input)
        output.push_back(tolower(c));
}
//MyParser::convertToUpperCase
void MyParser::convertToUpperCase(const std::string &input, std::string &output)
{
    output = "";
    for (auto c : input)
        output.push_back(toupper(c));
}
//MyParser::MyParser
MyParser::MyParser()
{
}
//MyParser::~MyParser
MyParser::~MyParser()
{
}
//MyParser_t& utils::getParser()
MyParser_t& utils::getParser()
{
    static MyParser_t p = std::unique_ptr<MyParser>(new MyParser());
    return p;
}
```

On obtient le résulat suivant après l'exécution des tests:

```bash
> cd build && cmake -DCMAKE_BUILD_TYPE=Debug ..
## truncated ##
build> cmake --build . --config Debug
## truncated ##
build> ../bin/Debug/toolset_test

==========] Running 5 tests from 2 test suites.
[----------] Global test environment set-up.
[----------] 2 tests from ParserTest
[ RUN      ] ParserTest.Parser_LowerSingleLetter
[       OK ] ParserTest.Parser_LowerSingleLetter (0 ms)
[ RUN      ] ParserTest.Parser_UpperSingleLetter
[       OK ] ParserTest.Parser_UpperSingleLetter (0 ms)
[----------] 2 tests from ParserTest (0 ms total)

[----------] 3 tests from UniqueParserTest
[ RUN      ] UniqueParserTest.Parser_UniqueParserIsUnique
[       OK ] UniqueParserTest.Parser_UniqueParserIsUnique (0 ms)
[ RUN      ] UniqueParserTest.Parser_UniqueParserLowerSingleLetter
[       OK ] UniqueParserTest.Parser_UniqueParserLowerSingleLetter (0 ms)
[ RUN      ] UniqueParserTest.Parser_UniqueParserUpperSingleLetter
[       OK ] UniqueParserTest.Parser_UniqueParserUpperSingleLetter (0 ms)
[----------] 3 tests from UniqueParserTest (0 ms total)

[----------] Global test environment tear-down
[==========] 5 tests from 2 test suites ran. (0 ms total)
[  PASSED  ] 5 tests.
```

Le refactoring et l'ajout de la fonction se sont bien déroulés et l'ensemble des tests unitaires définis depuis le début permettent de controler la qualité du code tout au long de la production. Ce qui me permet d'insister sur un aspect intéressant de l'approche TDD : les tests ne sont pas jettables --- on peut les faire évoluer, comme tout à l'heure avec l'ajout du namespace, mais ce serait dommage de les supprimer car il permettent de contrôler la qualité du code.

## Conclusion

Le TDD est une approche et pas une technique toute faite. Ce qui signifie qu'il faut s'exercer sur des projets avec rigueur et patience. Plus on s'exerce, plus on a de bons réflexes.

Même si les styles de programmation et les langages varient et qu'il est difficile de faire des généralités, on peut quand même définir quelques bonnes pratiques, valables pour tout type de projet. J'en cite 3 :
- les tests doivent êtres simples et exprimables en langage naturel
- on doit absolument s'interdire de faire évoluer le code sans avoir défini les tests qui permettront de valider la production
- il faut diversifier au maximum l'objet des tests (ne pas tester les mêmes choses) --- ce qui sera possible en envisageant le maximum de cas d'usage possible. Le lecteur pourra consulter les ouvrages sur le sujet pour se faire une idée plus complète des [méthologies TDD][9].

Le code utilisé dans le post est disponible [ici][8]. J'attends vos commentaires...

## Resources

- [CMake.org][1]
- [Git : official Website][2]
- [GitHub.com/google/googletest][4]
- [Crascit.com : cmake-gte][5]
- [docs.microsoft.com : CMake projects in visual studio][7]
- [Wikipedia : Test-driven development][9]
- [Kanmeugne's Blog : Introduction au TDD (II) -- le cycle vertueux (code source)][8]

[1]: https://cmake.org/cmake/help/v3.8/ "CMake est un système de construction logicielle multiplateforme"
[2]: https://git-scm.com/ "Git est un logiciel de gestion de versions décentralisé"
[4]: https://github.com/google/googletest "GoogleTest is Google’s C++ testing and mocking framework"
[5]: https://crascit.com/2015/07/25/cmake-gte "Crascit : Building GoogleTest and GoogleMock directly in a CMake project"
[6]: /posts/introduction-tdd-googletest "Introduction au TDD (I) : Mise en place avec googletest"
[7]: https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio "CMake est devenu de plus en plus intégré à Visual Studio au cours des dernières versions"
[9]: https://en.wikipedia.org/wiki/Test-driven_development "Wikipedia : Le Test-Driven Development (TDD), ou développement piloté par les tests en français, est une méthode de développement de logiciel qui consiste à concevoir un logiciel par petites étapes, de façon progressive, en écrivant avant chaque partie du code source propre au logiciel les tests correspondants et en remaniant le code continuellement."
[10]: /posts/tdd-cycle-vertueux/ "Introduction aux TDD: Le cycle vertueux"
[8]: https://github.com/kanmeugne/cppexperiments/releases/tag/starting-the-cycle "The source code of this tutorial is forkable from here"









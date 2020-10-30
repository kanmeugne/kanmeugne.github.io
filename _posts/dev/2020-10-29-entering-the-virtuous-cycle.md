---
layout: page
sidebar: right
title:  "Setting Up TDD with googletest (2/2)"
teaser: "Entering the virtuous cycle"
categories:
    - software development
tags:
    - cmake
    - c++
    - tdd
image:
    thumb: SFMLCMAKE-thumb.png
---

## Part 2/2 : Red, Green, Refactor

Rappelons le [mindset du developpeur TDD][9] :
1. écrire des petits tests
2. s'assurer que les tests échouent dans un premier temps
3. écrire le code qui permet passer le test
4. s'assurer que le test passe
5. nettoyer|réfactorer le code
6. s'assurer que le test passe toujours
7. retourner à l'étape 1.

Revenons au contenu de notre lib. Pour rappel, elle contient un parseur capable de réaliser les opérations suivantes:
- *convertToLowerCase* : convertit un mot ou une phrase en minuscule
- *convertToUpperCase* : convertit un mot ou une phrase en majuscule

On peut déjà définir notre premier test unitaire qui échouera.

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

`TEST` est une macro googletest qui permet de définir un test unitaire, et `ASSERT_EQ` une autre MACRO qui permet de tester si 2 variables ont la même valeur (voir la [documentation de googletest][4] pour plus d'infos sur les macros disponibles). On notera la déclaration de la classe `ParserTest` (dérivée de `testing::Test`) qui permet de regrouper les tests par thématique --- comme on le verra plus bas, ce mécanisme de classe permet aussi de définir des fixtures. 

Le test défini ci-dessus est simple (une assertion) avec un objectif exprimable en langage naturel : *le parseur doit transformer la lettre L (majuscule) en la lettre l (minuscule)*. Produisons maintenant qui permettra de le valider.


### Remember : tests should fail first

Pour l'instant le test échoue car la classe `MyParser` n'est pas vraiment définie. En écrivant le strict minimum dans `MyParser.h` pour que la compilation fonctionne,

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

on a un message plus conventionnel :

```bash
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from ParserTest
[ RUN      ] ParserTest.Parser_LowerSingleLetter
/home/patrick/Documents/Projects/cppexperiments/tddwithgtest/tests/src/main.cpp:15: Failure
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

Pour que le test passe il faut produire l'implémentation de la fonction `MyParser::convertToLowerCase` dans `MyParser.cpp`. Pour faire simple, disons que la fonction parcourt la chaine de caractères, transforme en minuscules tous les caractères et les rajoute dans la variable de sortie.

```c++
//MyParser::convertToLowerCase
void MyParser::convertToLowerCase(const std::string &input, std::string &output)
{
    output = "";
    for (auto c : input)
        output.push_back(tolower(c));
}
```

Cette fois-ci le test passe. On a produit le code qui valide notre premier test unitaire !

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

Ecrivons notre deuxième test pour la deuxième fonction (on aurait pu aussi ajouter plus de tests sur la première fonction), 

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

et ensuite les modifications de code qui nous permettent de valider le test. Tout le fichier `./toolset/include/MyParser.h`,

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

puis le fichier `./toolset/src/MyParser.cpp`,

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

On laissera le soin au lecteur de faire d'autre tests unitaires pour compléter les fonctionnalités de la bibliothèque. La logique est toujours la même : red (le test échoue d'abord), green (le test passe) et éventuellement refactor pour bien struturer le code.

On peut aussi faire directement du refactoring, dans ces cas on profite des tests déjà définis pour controler la qualité du code. Il ne s'agit plus seulement de définir de nouvelles fonctionnalités, mais aussi de restructurer le code et s'assurer que les tests passent toujours. 

Pour l'exemple, essayons un refactoring en deux étapes :
1. création d'un namespace `utils`, qui contiendra les définitions de la classe `MyParser`.
2. jouter d'une fonction dans la bibliothèque qui renvoie une instance unique de `MyParser`

### Refactoring 1 : adding a namespace

Pour la première étape, modifions tout d'abord les tests qui doivent valider le code.


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

Bien évidemment, ces tests échouent dans un premier temps puisque `MyParser.h` n'est pas défini dans un namespace. Les modifications suivantes vont permettre de les valider : sur le fichier `./toolset/include/MyParser.h`, 
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

puis le fichier `./toolset/src/MyParser.cpp`, dans lequel il suffira de rajouter la ligne ci-dessous :

```c++
using namespace utils;
```
Les tests refonctionnent, refactoring réussi, en toute sérénité. Maintenant voyons pour la deuxième étape.

### Refactoring 2: use a unique pointer

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

`UniqueParserTest` définit une fixture `UniqueParserTest::_parser` qui est initialisée dans le constructeur de la classe.

Les tests `UniqueParserTest` vont permettre de valider, d'une part, que la fonction `utils::getParseur` retoune toujours la même instance, et d'autre part, que toutes les fonctionnalités du parseur sont bien présente par cette instance.

On définit les nouveaux tests suivants (avec la MACRO `TEST_F` pour exploiter les fixtures) :

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
On notera l'utilisation d'un type spécial `MyParser_t` -- à définir dans le code à produire -- pour stocker l'instance unique de `MyParser`.

Bien évidemment, les tests échouent dans un premier temps, puis (après plusieurs essais), les modifications suivantes permettent de valider les tests : dans le fichier `MyParser.h`, 

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

puis dans le fichier `MyParser.cpp`,

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
build> cmake --build . --config Debug
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

Le refactoring et l'ajout de la fonction se sont bien déroulés. Les tests unitaires définis au début de l'opération ont permis de controler la qualité du code tout au long de l'opération.

## Conclusion

Le TDD est une approche et pas une technique toute faite. Ce qui signifie qu'il faut s'exercer sur des projets avec rigueur. Plus on s'exerce, plus on a de bons réflexes. Même si les styles de programmation et les langages peuvent variés, il y a quand même quelques règles générales, valable pour tous types de projet. J'en retiens 3 :
- les tests doivents êtres simples et exprimables en langage naturel
- on doit absolument s'interdire de faire évoluer le code sans avoir défini les tests qui permettront de valider la production.
- il faut diversifier au maximum l'objet des tests (ne pas tester les même choses).

Le lecteur pourra consulter les ouvrages sur le sujet pour se faire une idée plus complète des [méthologies TDD][9].



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
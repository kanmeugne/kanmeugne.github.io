---
sidebar: right
title:  "Setting Up Your python Environment (II)"
teaser: "Les environnements virtuels"
categories:
    - software development
tags:
    - python
    - jupyter
    - ipython
    - programming
comments: true
author: kanmeugne
---

Dans un [article][6] précédent, j'expliquais en 3 étapes comment mettre en place un environnement de programmation en Python. Il existe plusieurs options d'installation en réalité ([anaconda][1], [winpython][2], etc.) mais j'ai privilégié la plus *bas niveau* ⏬.

La [procédure][6] que j'ai présentée suppose l'utilisation d'un système Linux. J'ai insisté sur le fait qu'il ne s'agissait pas d'une contrainte -- bien au contraire -- et ce pour au moins 2 raisons :

1. Premièrement, Linux est très populaire auprès des développeurs - le débutant pourra donc automatiquement profiter d'une importante communauté d'entraide 💃.
2. Deuxièmement, Windows -- qui est de loin l'OS le plus populaire tout court -- propose des sous-systèmes Linux natifs dans ses dernières versions. Il est donc très facile de travailler sous linux aujourd'hui (encore plus que par le passé) quelque soit la version récente de Windows installée sur sa machine (pour les utilisateurs de MacOs, l'expérience montre que les procédures d'installation -- au moins à partir d'un terminal -- sont quasi similaires).

![Setting up environment 2](/images/ralston-smith-sEgodrJdMGw-unsplash.jpg){: width="500"}
_Photo by [Ralston Smith](https://unsplash.com/@ralstonhsmith?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)._

Dans cet article, nous allons un peu plus loin dans l'organisation de l'espace de travail du developpeur `python` avec la mise en place d'**environnments virtuels**.

## Pourquoi utiliser un environnement virtuel ?

Le monde des développeurs est un monde complexe, plein de contrariétés et d'épreuves.

Il peut arriver par exemple :
- que vous ayez besoin d'une version bien précise de l'interpréteur `python` -- différente de la version installée sur votre système -- parceque la `lib X` que vous convoitez ne marche qu'avec cette version-là !
- que, bien que vous utilisiez la version la plus récente de la `lib X` dans votre code `Y`, vous ayez besoin d'une version plus ancienne de la `lib X` pour qu'un autre bout de code `Z` -- que vous avez eu tant de mal à développer -- continue de fonctionner sur votre machine ! 
- que vous ayez envie -- et c'est votre droit le plus absolu -- d'isoler vos projets `python` pour avoir une bonne vision des dépendances et des `lib` utilisées. 

Pour faire simple, vous pouvez être confronté à deux cas de figure : 
1. *cohabitation* : vous avez besoin de faire cohabiter plusieurs versions d'interpreteurs `python` ou de `lib` `python`
2. *isolement* : vous voulez isoler vos projets pour avoir une bonne visibilité sur les `lib` et les dépendances nécessaires pour votre projet. 

Si vous vous retrouvez dans l'un ces deux cas de figure, vous avez certainement besoin d'utiliser un **environnement virtuel**. 

## Que faut-il installer pour utiliser un environnement virtuel ? 

Nous allons considérer que nous sommes dans les configurations de [l'article][6] cité à l'introduction -- si vous ne l'avez pas lu, faites-le et revenez vite 😼.

Commencez par installer [`virtualenv`][3] et [`virtualenvwrapper`][4] qui sont des `lib python` qui permettent :
- de créer un environnement de dev isolé du reste du système ([`virtualenv`][3])
- de gérer les environnements virtuel depuis le terminal ([`virtualenvwrapper`][4])

```shell
pip install virtualenv virtualenvwrapper
```

[`virtualenvwrapper`][4] créé un programme -- `virtualenvwrapper.sh` -- qui doit s'exécuter à chaque début session du terminal pour définir les commandes permettant de gérer les environnements virtuels. Assurez-vous de bien le localiser et de l'exécuter dans votre `~/.bashrc` pour que les commandes soient créées automatiquement à chaque session.

Pensez aussi à déclarer le dossier dans lequel les environnements virtuels seront créés `WORKON_DIR`.

Ci-dessous, un exemple de configuration.

```shell
# ~/.bashrc (ou ~/.zshrc)

# interpréteur par defaut
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3 

# les environnements virtuels sont crées ici
export WORKON_HOME=~/.virtualenvs 

## generer les commandes pour gérer les environnements virtuels
source ~/.local/bin/virtualenvwrapper.sh

# pour utiliser l'environnement virtuel dès sa création
export PIP_RESPECT_VIRTUALENV=true
```

> 💁 Pour plus d'options de configuration, bien vouloir consulter la [documentation officielle][4].
{: .prompt-tip }

Voilà, vous êtes prêts à tester les environnements virtuels.

## Comment utiliser un environnement virtuel ?

>  ⚠ Le tutoriel est basé sur [`virtualenv`][3] `20.2.2`!
{: .prompt-warning }

```bash
$ pip show virtualenv 
Name: virtualenv
Version: 20.2.2
Summary: Virtual python Environment builder
...
```

1. Premièrement, il faut en créer un... Et pour cela, vous devez utiliser la commande `mkvirtualenv`. Nous allons utiliser `mkvirtualenv` pour créer un environnement virtuel que nous allons appeler `myenv`.

```bash
$ mkvirtualenv --python 3 myenv
created virtual environment CPython3.8.10.final.0-64 in 200ms
 ...
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myenv/bin/predeactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myenv/bin/postdeactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myenv/bin/preactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myenv/bin/postactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myenv/bin/get_env_details

(myenv) $ 
```
Vous devriez obtenir l'équivalent des logs ci-dessus. Remarquez que le prompt a légèrement changé (si tout s'est bien passé). Vous avez maintenant `(myenv)` avant l'invite (cf. ligne 10).

&Agrave; partir de maintenant, on travaille dans un espace virtuel -- toutes les installations de lib se feront dans cet espace uniquement, et non sur l'ensemble du système.

On peut vérifier que l'espace nouvellement créé est quasi vierge et que très peu de `lib` sont pré-installées (juste de quoi installer d'autres `lib` 😉)

```bash
(myenv) $ pip list
Package    Version
---------- -------
pip        22.1
setuptools 46.1.3
wheel      0.34.2
```

Installons le prompt [`ipython`][5] dans notre environnement virtuel, histoire de le remplir un peu... Rien de très compliqué, il s'agit exactement des mêmes commandes que d'habitude -- `pip install ipython` -- la seule différence étant que l'installation se fait uniquement dans l'environnement virtuel.

```bash
(myenv) $ pip install ipython
Collecting ipython
  Using cached ipython-8.3.0-py3-none-any.whl (750 kB)
Collecting jedi>=0.16
  Using cached jedi-0.18.1-py2.py3-none-any.whl (1.6 MB)
Collecting traitlets>=5
  Using cached traitlets-5.2.1.post0-py3-none-any.whl (106 kB)
  ...
Successfully installed asttokens-2.0.5 ...
```

On peut constater que l'environnement est un peu plus chargé -- ce qui est tout a fait normal car [`ipython`][5] a été installé, avec plusieurs autres dépendances (nécessaires au fonctionnement de `ipython`).

```bash
(myenv) $ pip list
Package           Version
----------------- -----------
asttokens         2.0.5
backcall          0.2.0
decorator         5.1.1
executing         0.8.3
ipython           8.3.0
jedi              0.18.1
matplotlib-inline 0.1.3
parso             0.8.3
pexpect           4.8.0
pickleshare       0.7.5
pip               22.1
prompt-toolkit    3.0.29
ptyprocess        0.7.0
pure-eval         0.2.2
Pygments          2.12.0
setuptools        46.1.3
six               1.16.0
stack-data        0.2.0
traitlets         5.2.1.post0
wcwidth           0.2.5
wheel             0.34.2
```

Voilà! On peut maintenant utiliser [`ipython`][5] depuis notre environnement virtuel `myenv`.

```python
(myenv) ipython
Python 3.8.10 (default, Mar 15 2022, 12:22:08) 
Type 'copyright', 'credits' or 'license' for more information
IPython 8.3.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: print("coucou")
coucou

In [2]: 

```

Vous pouvez sortir de l'environnement virtuel -- commande `deactivate` -- et constater que vous n'avez plus accès à [`ipython`][5] (sauf si vous l'aviez sur tout le système avant bien sûr. Si c'est le cas, désinstallez-le avant de suivre ce tutoriel). 

```bash
(myenv) $ deactivate

$ ipython
bash: command not found: ipython
```

## Comment sauvegarder mon environnement virtuel ?

Un des interêts des environnements virtuels c'est d'avoir une vision claire des dépendances nécessaires pour son code `python`. On peut ainsi reproduire son environnement de travail en toute sérénité et n'importe quand. 

Pour sauvegarder et cloner son environnement virtuel, on peut procéder de la manière suivante :

1. je me connecte à l'environnement que je souhaite sauvegarder :

```zsh
$ workon myenv

(myenv) $ 
```

1. je sauvegarde l'état de l'environnement dans un fichier texte, grâce notemment à la commande `pip freeze`.

> A la différence de `pip list`, la commande `pip freeze` affiche les dépendances dans un format directement exploitable pour l'installation. 
{: .prompt-tip }

```bash
(myenv) $ pip freeze
asttokens==2.0.5
backcall==0.2.0
decorator==5.1.1
executing==0.8.3
ipython==8.3.0
jedi==0.18.1
matplotlib-inline==0.1.3
parso==0.8.3
pexpect==4.8.0
pickleshare==0.7.5
prompt-toolkit==3.0.29
ptyprocess==0.7.0
pure-eval==0.2.2
Pygments==2.12.0
six==1.16.0
stack-data==0.2.0
traitlets==5.2.1.post0
wcwidth==0.2.5

(myenv) $ pip freeze > requirements.txt

(myenv) $ ls *.txt
requirements.txt
```
3. enfin, je clone mon environnement grâce au fichier de sauvegarde `requirements.txt` (pour l'exemple, on supprime `myenv` au préalable, avec la commande `rmvirtualenv`)

```bash
(myenv) $ deactivate

$ rmvirtualenv myenv
Removing myenv...

$ mkvirtualenv --python 3 -r requirements.txt myclonenv
created virtual environment CPython3.8.10.final.0-64 in 112ms
...
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myclonenv/bin/predeactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myclonenv/bin/postdeactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myclonenv/bin/preactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myclonenv/bin/postactivate
virtualenvwrapper.user_scripts creating ~/.virtualenvs/myclonenv/bin/get_env_details
Collecting asttokens==2.0.5
  Using cached asttokens-2.0.5-py2.py3-none-any.whl (20 kB)
Collecting backcall==0.2.0
  Using cached backcall-0.2.0-py2.py3-none-any.whl (11 kB)
...
Installing collected packages: six, asttokens, ...

$ (myclonenv) pip list
Package           Version
----------------- -----------
asttokens         2.0.5
backcall          0.2.0
decorator         5.1.1
executing         0.8.3
ipython           8.3.0
jedi              0.18.1
matplotlib-inline 0.1.3
parso             0.8.3
pexpect           4.8.0
pickleshare       0.7.5
pip               22.1.1
prompt-toolkit    3.0.29
ptyprocess        0.7.0
pure-eval         0.2.2
Pygments          2.12.0
setuptools        46.1.3
six               1.16.0
stack-data        0.2.0
traitlets         5.2.1.post0
wcwidth           0.2.5
wheel             0.34.2
```

Votre environnement est parfaitement cloné ! Et vous voilà initié à l'utilisation des environnements virtuels en `python`. 

> Très utile : 
> - l'aide sur la commande `mkvirtualenv` -- `mkvirtualenv --help`
> - les documentations officielles de [`virtualenv`][3] et [`virtualenvwrapper`][4] pour plus d'options.
{: .prompt-tip }

## Références

- [Anaconda.com][1]
- [Winpython.github.io][2]
- [Virtualenv.pypa.io][3]
- [Virtualenvwrapper.readthedocs.io][4]
- [Ipython.org][5]
- [Kanmeugne's Blog : Setting Up Your Python Environment][6]

[1]: https://www.anaconda.com/
[2]: https://winpython.github.io/
[3]: https://virtualenv.pypa.io/en/latest/ "cliquez ici pour consulter la documentation officielle de virtualenv"
[4]: https://virtualenvwrapper.readthedocs.io/en/latest/ "cliquez pour consulter la documentation officielle de virtualenvwrapper"
[5]: https://ipython.org/documentation.html "cliquez ici pour consulter la documentation officielle de ipython"
[6]: /posts/setting-up-your-python-environment "Lire l'article "Setting Up Your python Environment (I)""
---
sidebar: right
title:  "Setting Up Your Python Environment II"
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

Dans un aricle précédent, j'expliquais en 3 étapes comment mettre en place un environnement de programmation en Python. Il existe plusieurs options d'installation en réalité (anaconda, winpython, etc.) mais j'ai privilégié la plus "bas niveau".

La procédure que j'ai présentée préssupose l'utilisation d'un système Linux et il ne s'agit de pas d'une contrainte bien au contraire, et pour au moins 2 raisons. Premièrement, Linux est très populaire auprès des développeurs - le débutant pourra donc automatiquement profiter d'une importante communauté d'entraide. Deuxièmement, Windows - qui est de loin l'OS le plus populaire tout court - propose des sous-systèmes Linux natifs dans ses dernières versions. Il est donc très facile de travailler sous linux aujourd'hui (encore plus que pas le passé) quelque soit l'OS installé sur sa machine (pour les utilisateurs de MacOs, l'expérience montre que les procédures d'installation - au moins à partir d'un terminal - sont quasi similaires).

Dans cet article, nous allons un peu plus loin dans l'organisation de l'espace de travail du developpeur python avec la mise en place d'environnments virtuels.

## Pourquoi un environnement virtuel ?

Le monde des développeurs est un monde complexe, plein de contrariétés et d'épreuves. Il peut arriver par exemple :
- que vous ayez besoin d'une version bien précise de l'interpréteur python -- différente de la version installée sur votre système -- parceque la `lib X` que vous convoitez ne marche qu'avec cette version-là !
- que, bien que vous utilisiez une version plus récente de la `lib X` dans l'`app Y`, vous ayez besoin d'une version plus ancienne de la `lib X` pour que votre `app Y` -- que vous avez eu tant de mal à développer -- continue de fonctionner sur votre machine ! 
- que vous ayez envie, et c'est votre droit le plus strict, d'isoler vos projets python pour avoir une bonne visibilité des dépendances et des lib utilisées. 

Pour faire simple, vous pouvez confronter à deux cas de figure : 
- cohabitation : vous avez besoin de faire cohabiter plusieurs d'interpreteur ou de lib python
- isolement : vous voulez isoler vos projets pour avoir une bonne visibilité sur les libs et les dépendances nécessaires. 

Si vous vous retrouvez dans l'un ces deux cas de figure, vous avez certainement besoin d'utiliser un environnement virtuel. 

## Installation et mise en oeuvre

Que faut-il installer pour mettre en oeuvre un environnement virtuel ? et comment ça se passe concrètement ? C'est ce que nous allons voir maintenant. Nous allons considérer que nous sommes dans les configurations de l'article [setting up your python environment](/dev/2021-05-03-setting-up-your-python-environment.md).

Il faut tout d'abord installer `virtualenv` et `virtualenvwrapper` qui sont des lib python qui permettent :
- de créer un environnement de dev isolé du reste du système (virtualenv)
- de gérer les environnements virtuel depuis le terminal (`virtualenvwrapper`)

```python
pip install virtualenv virtualenvwrapper
```

`virtualenvwrapper` créé un programme -- `virtualenvwrapper.sh` -- qui doit s'exécuter à chaque début session du terminal pour définir les commandes permettant de gérer les environnements virtuels. Assurez-vous de bien le localiser et de l'exécuter dans votre `~/.bashrc`. Pensez aussi à déclarer le dossier dans lequel les environnements virtuels seront créés `WORKON_DIR`. Ci dessous, un esemple de configuration.

```bash
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

Je renvoie le lecteur à la documentation officielle pour avoir les instructions les plus à jour.


[1]: https://www.howtogeek.com/790062/how-to-install-bash-on-windows-11/
[2]: https://www.python.org "see the available version"
[3]: https://docs.python-guide.org/starting/install3/linux/
[4]: https://www.guru99.com/python-ide-code-editor.html#:~:text=IDLE%20%28Integrated%20Development%20and%20Learning%20Environment%29%20is%20a,can%20be%20used%20on%20Windows%2C%20macOS%2C%20and%20Unix
[5]: https://jupyter.org/install
[6]: https://jupyter.org/
[7]: https://ipython.org/
[8]: https://code.visualstudio.com/docs
[9]: https://www.sublimetext.com/
[10]: https://www.jetbrains.com/pycharm/
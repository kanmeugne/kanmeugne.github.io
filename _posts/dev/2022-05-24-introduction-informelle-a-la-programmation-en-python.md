---
sidebar: right
title:  "Introduction informelle à la programmation en Python"
teaser: "Pour les déutants pressés"
categories:
    - software development
tags:
    - python
    - programming
    - paradigm
comments: true
author: kanmeugne
---

Python est souvent présenté comme :
- un langage interprété -- par contraste avec des langages compilés comme java, c, ou c++
- un langage non typé et facile d'utilisation -- ce qui le différencie des langages où il faut déclarer les types de variables utilisées
- un langage orienté-objet -- par contraste avec les langages impératifs, fonctionnels, etc.

Même s'il s'agit de détails importants pour décrire le langage en lui-même, toutes ces notions sont difficiles à comprendre et encombrantes pour le débutant en programmation. Pour apprendre à écrire un programme en python, le plus important à mon avis est d'avoir le bon mindset. Les différences avec les autres langages se comprennent avec la pratique et avec l'expérience. C'est en quelque sorte, ce que j'essaie de montrer dans cette série d'articles qui se veut être une initiation "informelle" au langage python. L'idée est de poser les bases de la programmation en python de manière très pratique, en présentant le strict nécessaire de la philosophie du langage pour un développeur débutant. Est-ce vraiment possible? Vous me direz...

![Setting up environment 2](/images/william-bayreuther-1ZWQnCVJkm8-unsplash.jpg){: width="500"}
_Photo by [William Bayreuther](https://unsplash.com/@wbayreuther?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)._


Cette série sera organisée de la manière suivante : 
- Dans le premier article, j'expliquerai la différence -- à l'exécution -- d'un programme python par rapport à d'autres langages. Il s'agit d'un petit détail, mais qui en dit long sur l'état d'esprit qu'il faut avoir lorsqu'on pratique le langage python.
- Dans la seconde section, je présenterai ce que j'appelle le "why-first", qui se traduirait en français par "quoi-d'abord". J'attirerai l'attention du débutant sur l'importance de formuler ce qu'on veut faire avec des verbes d'action simples avant de le coder. 
- Enfin, je tenterai une démystification du bug -- terreur du débutant -- qui est pourtant une excellente façon d'apprendre.

> Cet article est une réflexion personnelle. Désolé d'avance si je choque les puristes.
{: .prompt-warning }

## A quoi sert un programme informatique ?




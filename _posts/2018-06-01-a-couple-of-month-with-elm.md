---
layout: post
title: "A couple of month with elm"
excerpt: "Depuis quelques années notre métier de développeur•se a evolué tant en terme de framework (Il en sort quasi toutes les semaines ;) ) mais aussi en termes de languages. On voit fleurir des languages typés destiné au frontend comme par exemple [reasonML](https://reasonml.github.io/), [Elm](http://elm-lang.org/). Mais celui dans lequel je me suis plongé c'est [Elm](http://elm-lang.org/)."
date: 2018-06-01 08:00:00 +0100
categories: Experience Elm
tags: elm
---

Depuis quelques années notre métier de développeur•se a evolué tant en terme de framework (Il en sort quasi toutes les semaines ;) ) mais aussi en termes de languages. On voit fleurir des languages typés destiné au frontend comme par exemple [reasonML](https://reasonml.github.io/), [Elm](http://elm-lang.org/). Mais celui dans lequel je me suis plongé c'est [Elm](http://elm-lang.org/).

# Elm

Le language [Elm](http://elm-lang.org/) est un language fortement [typé](https://fr.wikipedia.org/wiki/Typage_fort), [fonctionnel](https://fr.wikipedia.org/wiki/Programmation_fonctionnelle) et immutable. On passe par une phase de compilation pour avoir en résultat un fichier javascript qui pourra être interprêté par tous les navigateurs.

## Le language

Alors je vous avouerai qu'au départ le language est un peu déroutant surtout pour ceux et celles qui sont vraiment orientés frontend et qui mangent du javascript tous les jours. Il n'y a pas de mots clés pour assigner une valeur à une variable et d'ailleur en fait il n'y a pas de variable car en fait dans [Elm](http://elm-lang.org/) tout est fonction et donc en fait on crée une fonction qui retourne 42.

```elm
 -- ceci est en fait une fonction qui retourne la valeur 42.

 everything = 42

```

Vu que tout est fonction en Elm on se retrouve souvent devant ce type de code :

```Elm

add : Number -> Number -> Number
add a b =
  a + b

```

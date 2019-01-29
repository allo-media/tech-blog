---
layout: post
title:  "Un pas de plus dans Elm."
excerpt: "Ma compréhension sur le langage a fait un bon en avant, quand j'ai réussi à comprendre les signatures de Maybe.map, Task.map, Decode.map ."
date:   2019-01-29 23:07:00 +0100
categories: Elm
tags: elm map
---


#  Un pas de plus dans Elm.

Ma compréhension sur le langage a fait un bon en avant, quand j'ai réussi à comprendre les signatures de Maybe.map, Task.map, Decode.map.

```elm
Maybe.map : (a -> b) -> Maybe a -> Maybe b
```

Nous allons prendre comme exemple la première fonction *(Maybe.map)*.
Si nous nous reportons à sa signature, on voit que le premier paramètre est une fonction symbolisé par les `()`, et le second paramètre est un `Maybe a` et cela retourne un `Maybe b`.

## fonction .map   

.map n'est ni plus ni moins qu'une fonction de transformation qui prend en premier paramètre la fonction de transformation et en second paramètre la donnée que vous souhaitez transformer pour retourner le résultat de cette transformation.

```
Maybe.map : (a -> b) -> Maybe a -> Maybe b
```

| Nom de la fonction | Premier paramètre | Second paramètre | Résultat de la transformation |
|:-:|:-:|:-:|:-:|
| Maybe.map            | (a -> b)| Maybe a          | Maybe b |

Nous allons retrouver toute la logique de transformation dans le premier paramètre. En le décomposant on peut voir que c'est une fonction qui prend en premier paramètre, le second paramètre `Maybe a` de `Maybe.map` et renverra le résultat de la transformation à savoir `Maybe b`.

Comme exemple je vais tenter de transformer mes fruits en jus du fruits. On part d'un nombre fruits incertains (je peux rentrer bredouille de la cueillette) pour ensuite le transformer en jus de fruits.


```
freshFruitJuice : Maybe Int -> Maybe Float
freshFruitJuice fruit =
    Maybe.map (\f -> toFloat f * 0.1) fruit
```

Nous avons la possibilité aussi de passer plusieurs paramètres à notre fonction de transformation en utilisant map2, map3, map4

```
freshFruitJuice : Maybe Int -> Maybe Int -> Maybe Int -> Maybe Float
freshFruitJuice orange lemon carrots =
    Maybe.map3 (\o l c -> (toFloat o + toFloat l + toFloat c) * 0.1) orange lemon carrots
```

Une fois que vous aurez compris cette partie là vous verrez que les Decode.map, Task.map, etc.. vous serrons beaucoup moins obscure.

Du coup maintenant si nous prenons un cas plus concret avec les decodeurs par exemple. Nous souhaitons transformer un payload json en User.
Nous avons utilisé `Decode.map3`

```
Decode.map3 : (a -> b -> c -> value) -> Decoder a -> Decoder b -> Decoder c -> Decoder value
```

afin d'avoir en paramètre l'id, le firstname, et lastname et de créer un record de type User.

```
type alias User =
    { id : String
    , firstname : String
    , lastname : String
    }


decode : Decoder User
decode =
  Decode.map3 (id firstname lastname -> User id firstname lastname)
    (Decode.field "id" Decode.string)
    (Decode.field "firstname" Decode.string)
    (Decode.field "lastname" Decode.string)

```

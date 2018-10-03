---
layout: post
title:  "Text2num version 1.0.0 released!"
excerpt: "There already existed some python packages to convert numbers written in english into python numbers or their decimal
digit representation, but there was nothing available for the french language. That's why we developped this library and shared it with the community."
date:   2018-10-02 12:53:00 +0100
categories: python
tags: python NLP
---

The output of speech-to-text systems are entirely made of words, without punctuation or capitalization. This makes visual scanning for numbers quite cumbersome,
especially in transcriptions of real life dialogues as they also contain a lot of gibberish words — like « heu, ben, bah… » — and the syntax or
grammar is not always right. Besides that, text mining tools and techniques often, if not always, expect numbers to be in decimal digit representation.

That's why we decided to add a transformation pass to our speech-to-text engine in order to convert all spoken numbers into their digit spelling.

Here at Allo-Media, we are fond of Open Source Software. So we first looked at the state of the art of python libraries for parsing words into numbers. There were some for the english language, but we didn't find any for french. Therefore, we decided to build our own library and contribute it back to the community.

Here are two samples of what you can expect from it:

```python
>>> from text_to_num import text2num
>>> text2num('nonante-cinq')
95

>>> text2num('mille neuf cent quatre-vingt dix-neuf')
1999

>>> text2num("cinquante et un million cinq cent soixante dix-huit mille trois cent deux")
51578302

>>> text2num('mille mille deux cents')
AssertionError
```

```python
>>> from text_to_num import alpha2digit
>>> sentence = (
...         "Huit cent quarante-deux pommes, vingt-cinq chiens, mille trois chevaux, "
...         "douze mille six cent quatre-vingt-dix-huit clous.\n"
...         "Quatre-vingt-quinze vaut nonante-cinq. On tolère l'absence de tirets avant les unités : "
...         "soixante seize vaut septante six.\n"
...         "Nombres en série : douze quinze zéro zéro quatre vingt cinquante-deux cent trois cinquante deux "
...         "trente et un.\n"
...         "Ordinaux: cinquième troisième vingt et unième centième mille deux cent trentième.\n"
...         "Décimaux: douze virgule quatre-vingt dix-neuf, cent vingt virgule zéro cinq ; "
...         "mais soixante zéro deux."
...     )
>>> print(alpha2digit(sentence))
```

```
842 pommes, 25 chiens, 1003 chevaux, 12698 clous.
95 vaut 95. On tolère l'absence de tirets avant les unités : 76 vaut 76.
Nombres en série : 12 15 004 20 52 103 52 31.
Ordinaux: 5ème 3ème 21ème 100ème 1230ème.
Décimaux: 12,99, 120,05 ; mais 60 02.
```

As you see, we support decimal numbers as well as ordinal numbers.

The algorithm is quite robust and is based on the observation that big numbers are structured by powers of thousand in the language, each group following the same pattern hundreds-tens-units. It handles the alternate forms « *septante / soixante-dix* » and « *nonante / quatre-vingt-dix* » and offers a relaxed mode for making the dash ([1990 reform](https://fr.wikipedia.org/wiki/Rectifications_orthographiques_du_fran%C3%A7ais_en_1990#Les_modifications_apport%C3%A9es)) optional.

The project is hosted on GitHub : [https://github.com/allo-media/text2num](https://github.com/allo-media/text2num).

Enjoy!
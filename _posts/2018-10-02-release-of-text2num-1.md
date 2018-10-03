---
layout: post
title:  "Text2num version 1.0.0 released!"
excerpt: "There already existed some python packages to convert numbers written in English into Python numbers or their decimal
digit representation, but there was nothing available for the French language. That's why we developed this library and shared it with the community."
date:   2018-10-02 12:53:00 +0100
categories: python
tags: python NLP
---

The output of speech-to-text systems are entirely made of words, without punctuation or capitalization. This makes visual scanning for numbers quite cumbersome,
especially in transcriptions of real life dialogues as they also contain a lot of gibberish words — like « heu, ben, bah… » — and the syntax or
grammar is not always correct. Besides that, text mining tools and techniques often, if not always, expect numbers to be in decimal digit representation.

That's why we decided to add a transformation pass to our speech-to-text engine in order to convert all spoken numbers into their digit spelling.

Here at Allo-Media, we are fond of Open Source Software. So we first looked at the state of the art of [Python](http://www.python.org) libraries for parsing words into numbers. There was at least [one](https://pypi.org/project/word2number/) for the English language, but we didn't find any for French. Therefore, we decided to build our own library and contribute it back to the community.

We could have ported the [Word2number](https://pypi.org/project/word2number/) library, but it has some flaws:

- It is unable to detect by itself the bounds of a number expression;
- its algorithm is weak (ex: `w2n.word_to_num('hundred five fifty') == 550`);
- French has some pecularities like « *quatre-vingt-dix-neuf* » vs « *nonante-neuf* ».

So we started a linguistic parser from scratch that is able to identify numbers and correctly isolate contiguous ones in a sequence. Moreover, we wanted it to be able
to parse different flavors of french (e.g. *soixante-dix* and *septante* for 70, etc…).

If you are interested in linguistics, *septante* for 70 and *nonante* for 90 are used in Belgium, Switzerland, Luxembourg, Aosta Valley, Jersey French and to a lesser extend in French regions of Savoie, Franche-Compté and even sometimes in Lorraine and Provence (source [Wikipedia](https://fr.wikipedia.org/wiki/70_(nombre)#Linguistique)). The rest of the French speaking world uses respectively *soixante-dix* and *quatre-vingt-dix*. The usage of *huitante* and *octante* instead of *quatre-vingts* is [more restricted yet](https://fr.wikipedia.org/wiki/80_(nombre)#Huitante).

Here are two samples of what you can expect from it:

```python
>>> from text_to_num import text2num
>>> text2num('quatre-vingt-quinze')
95

>>> text2num('nonante-cinq')
95

>>> text2num('mille neuf cent quatre-vingt dix-neuf')
1999

>>> text2num("cinquante et un million cinq cent soixante dix-huit mille trois cent deux")
51578302

>>> text2num('cent cinq cinquante')
AssertionError
```

```python
>>> from text_to_num import alpha2digit
>>> alpha2digit('cent cinq cinquante')
'105 50'

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

The algorithm is quite robust and is based on the observation that big numbers are structured like a sum of **decreasing** powers of thousand in the language, each power of thousand being multiplied by a number from 1 (maybe omitted) to 999. The problem is thus "reduced" to recognizing powers of thousands (*mille, million, milliard*) and being able to parse numbers from 1 to 999.

Example: *trois millions cinq cent vingt-trois mille deux cent quarante* -> 3 × 1 000 000 + 523 × 1000 + 240.

Parsing numbers between 1 and 999 is more difficult. The basic idea is that we expect between 0 and 9 hundreds, followed by a ten expression (*vingt, trente, …*) or none and some optional units (from 1 to 9) or extended units (from 1 to 19). The "hard" part is to detect illegal combinations and the end of the number.

As the needs arise, we may develop parsers for other languages on this base, including an robust English one with all the desired features.

If you are interested in more details or want to contribute, you can check the sources on [GitHub](https://github.com/allo-media/text2num) and the [contribution guide](https://text2num.readthedocs.io/en/stable/contribute.html).

If you just want to use it, it's just a `pip install text2num` away and the documentation is on [ReadTheDocs](https://text2num.readthedocs.io/).

Enjoy!
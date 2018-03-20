---
layout: post
title:  "From python to Go to Rust"
excerpt: "When looking for a new backend language, I naturally went from Python to Go (isn't what Google did?). But after only one week of Go, I realised that Go was only half of a progress. Better suited to my needs than Python, but too far away from the developer experience I enjoy when doing Elm in the frontend. So I gave Rust a try."
date:   2018-02-05 09:00:00 +0100
categories: point of view
tags: rust go elm
---

When looking for a new backend language, I naturally went from __Python__ to the new cool kid: __Go__. But after only one week of Go, I realised that Go was only half of a progress. Better suited to my needs than Python, but too far away from the __developer experience__ I was enjoying when doing Elm in the frontend. So I gave Rust a try.

## Away from Python,

For backend development, I've mainly been using Python 3 for the past three years. From admin scripts to machine learning to Flask/Django applications, I've done a lot of Python lately, but at some point, __it didn't felt right anymore__. Well, to be honest, it's not really at "some totally random point" that it started not to feel right anymore, it was when I started to enjoy programming with a strongly typed language: __Elm__.

I had the famous feeling "when it compiles it works", and once you've experienced that, __there is no way back__. You try stuff, you follow the friendly compiler error messages, you fix things, and then _tada_, it works!

Ok so, at this point I knew what I wanted from the "perfect" backend language:

1. __Static__ and __strong__ typing
2. Most of the stuff checked at __compile time__ (please, no exceptions!)
3. __No `null`__
4. __No mutability__
5. Handle __concurrency__ nicely

I see you coming: "hey, this is __Haskell__"! Yeah, I may write another blog post about it, but for me the problem with Haskell is not the language itself, it's what goes with it: elit mindset from the outside, poor documentation and little to no efforts to make it accessible for beginners.

"Hey, and what about Scala?!". What do you mean by Scala? The better Java? The functional programming language with scalaz? The Object Orienting Programming Functional language that may or may not fail at runtime with a `java.lang.NullPointerException` and needs a 4GB JVM running? I tried it some years ago and definitievely, this is a no go for me.

After discussing with a few people, I decided to give __Go__ a try. It has a __compiler__, __no exceptions__, no `null` (but __null values__) and can handle __concurrency__ nicely.

## Into Go,

I decided to rewrite an internal project that was already done in Python using Go. Just to get a feeling of the differences between the two.

First feeling: learning __Go__ was so easy. In __one evening__, I was able to compile a Proof Of Concept of the projet with basic features developped and some tests written. This was a very pleasant feeling, I was adding features very fast. The compiler messages were helpful, everything was fine.

And at some point, the tragedy started. I needed to add a field to some struct, so I just modified the struct and was ready to analyze the compiler messages to know where this struct was used in order to add the field where it was needed.

I compiled the code and … no error message. Everything went fine. But?! I just added a field to a struct, the compiler should say that my code is not good anymore because I'm not initializing the value where it should be!

The problem is that, not providing a value to a struct is not a problem in __Go__. This value will default to it's [zero value](https://tour.golang.org/basics/12) and everything will compile. This was the __show stopper__ for me. I realized that I __couldn't rely on the compiler__ to get my back when I was doing mistakes. At this point, I was wondering: why should I bother learning __Go__ if the compiler can't do much better than [Python and mypy](http://mypy-lang.org/)? Of course concurrency is much better with __Go__, but the downside of not being able to rely on the compiler was too much for me.

## Into Rust.

So __Go__ was not an option anymore as I realized that what I really needed was a __useful compiler__: a compiler that __should not rely on the fact that I know how to code__ (as it has been proven to be false a lot of times). That's why I took a look at __Rust__.

Rust was not my first choice because it advertises itself as a "system language", and I'm more of a web developer than a system one. But it had some very strong selling points:

- No `null` values but an `Option` type (checked at compile time)
- No `exceptions` but a `Result` type (checked at compile time)
- Variables are __immutable__ by default
- Built for concurrency in mind

I decided to rewrite the __same program__ than the one I did in Python and Go. The onboarding was a lot harder than with Go. As I did with Go, I tried to go cold turkey, but it was too hard: I needed some new concepts specific to Rust like __ownership__ or __lifetimes__ to understand the code I was seeing on StackOverflow. So I had no choice but to read the [Rust Book](https://doc.rust-lang.org/book/second-edition/), and it took me two weeks before I could start writing some code (remember that with Go it took me one evening…).

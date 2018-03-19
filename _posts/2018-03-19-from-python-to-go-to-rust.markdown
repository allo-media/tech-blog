---
layout: post
title:  "From python to Go to Rust"
excerpt: "When looking for a new backend language, I naturally went from Python to Go (isn't what Google did?). But after only one week of Go, I realised that Go was only half of a progress. Better suited to my needs than Python, but too far away from the developer experience I enjoy when doing Elm in the frontend. So I gave Rust a try."
date:   2018-02-05 09:00:00 +0100
categories: point of view
tags: rust go elm
---

When looking for a new backend language, I naturally went from Python to Go (isn't what Google did?). But after only one week of Go, I realised that Go was only half of a progress. Better suited to my needs than Python, but too far away from the developer experience I enjoy when doing Elm in the frontend. So I gave Rust a try.

## Away from Python

For backend development, I've mainly been using Python 3 for the past three years. From admin scripts to machine learning to Flask/Django applications, I've done a lot of Python lately, but at some point, __it didn't felt right anymore__. Well, to be honest, it's not really at "some totally random point" that it started not to feel right anymore, it was when I started to enjoy programming with a strongly typed language: __Elm__.

I had the famous feeling "when it compiles it works", and once you've experienced that, __there is no way back__. You try stuff, you follow the friendly compiler error messages, you fix things, and then _tada_, it works!

Ok so, at this point I knew what I wanted from the "perfect" backend language:

1. __Static__ and __strong__ typing
2. Most of the stuff checked at __compile time__ (please, no exceptions!)
3. __No `null`__
4. __No mutability__

I see you coming: "hey, this is __Haskell__"! Yeah, I may write another blog post about it, but for me the problem with Haskell is not language itself, it's what goes with it: elit mindset from the outside, poor documentation, no efforts to make it accessible for beginners and so on.

"Hey, and what about Scala?!". What do you mean by Scala? The better Java? The functional programming language with scalaz? The Object Orienting Programming Functional language that may or may not fail at runtime with a `java.lang.NullPointerException` and needs a 4GB JVM running? I tried it some years ago and definitievely, this is a no go for me.

---
layout: post
title: Baby's first Mathematica meta-tutorial
categories: [programming]
published: false
tags:
- mathematica
- haskell
- functional programming
---

[mathse-pitfalls]: https://mathematica.stackexchange.com/questions/18393/what-are-the-most-common-pitfalls-awaiting-new-users
[learnpython]: http://www.learnpython.org/
[lyah]: http://learnyouahaskell.com/chapters

Note: In this post, I shall refer to both Wolfram Mathematica (the symbolic computation program) and the Wolfram Language (the programming language in it) commonly as Mathematica.

### Why you might (not) want to read the rest

This post is meant to be an introductory guide to learning Mathematica, especially if your programming experience has been limited to object-oriented or imperative languages such as Java or C++.

This post is not meant to teach you how to write fast Mathematica code, but idiomatic Mathematica code, i.e., code which looks like Mathematica and not like a straight-forward syntax translation from \*insert favorite OO/imperative language here\*. Fortunately, idiomatic Mathematica code is often very fast because it relies heavily on using built-in functions which have been optimized well.

In case you are already familiar with functional programming (from say Lisp, Scala, OCaml etc.) feel free to jump to the section [Batteries are included so get off your bullock cart](#batteries-are-included-so-get-off-your-bullock-cart).

#### Too much freedom can be a bad thing (!)

Mathematica supports a large variety of programming styles (it is multi-paradigm). It has conventional looping constructs such as `do` and `for` and conditional statements such as `if`. Variables declared normally are mutable. At first glance, the lists appear to be equivalent to arrays or lists in other languages, apart from the strange syntax to access elements with `a[[n]]` instead of just `a[n]` and the one-based indexing.

These four features, when combined, can allow a beginner to write any simple program that they want to. This is bad for two reasons:

1. Since these features are available, you might not ask if there is another way to go about solving the problem at hand, instead sticking to solution you have already thought of.
2. As a consequence of 1., the code written is harder to read, reason about, debug and quite likely less performant than idiomatic code.

#### Why did I write this post...

While there are several excellent posts with tips for beginners, including this [Q & A thread][mathse-pitfalls] on Mathematica StackExchange (which you definitely should read and re-read), I think that none of them directly point out a direct way in which one can do so. So here is my (suggestion + grand claim + summary of the post) all rolled into one:

> Mathematica is a powerful multi-paradigm language. You can do a lot of wonderful things with it like symbolic manipulation, curve-fitting, fancy plotting etc. But as Uncle Ben used to say, with great power comes great responsibility. You can easily write bad/ugly/WTF Mathematica code. It is your responsibility towards yourself -- and those you will maintain your code after you leave -- to avoid doing so. 
> 
> You can (and should!) *learn* to avoid writing imperative-style code, especially if you are a beginner. One way to do so is by starting out with a pure, functional language like Haskell to understand the functional paradigm. Then you should proceed to write Mathematica code in the spirit of writing Haskell.

#### This post is not a Mathematica tutorial!

If you are familiar with a programming language, Mathematica's syntax will very likely be easy to grasp. The rest of this post assumes that you are familiar with basic Mathematica syntax (such as `:=` vs `=`) and are able to perform elementary tasks like writing a factorial function, plot `f(x)` vs `x` and are able to roughly understand the built-in documentation. This post's primary aim is to teach you how to learn Mathematica, not how to Mathematica.

On the other hand, I will talk a bit about Haskell. Since there are several great Haskell resources on the web, I will only briefly discuss the syntax and mumble a bit about types and skip over any advanced concepts. I hope that the discussion here is self-sufficient so that you understand the entire post without writing any Haskell yourself; however, I suggest you do learn a bit of Haskell -- equivalent to roughly the first 6 chapters of [Learn You a Haskell][lyah] -- if you want to actually "write Mathematica code in the spirit of writing Haskell".

### Purity: Programming as Mathematics

### Batteries are included so get off your bullock cart

Mathematica has a large number of built-in functions, some complex and daunting, some seemingly trivial and pointless. However, using these is key to clearer, concise and fast code. As a rule of thumb, when you are writing basic programs, it is a good idea to check if the functionality of your code (or a part of it) is already present in some built-in function.

### The world turns topsy-turvy

### Style and code clarity


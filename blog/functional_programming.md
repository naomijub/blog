---
title: Functional Programming
date: 2024-12-09
description: Insights on Functional Programming
keywords: "Functional Programming, Insights, Guide"
---

# Functional Programming

How can we start a functional programming-centred blog without talking about functional programming? Although I started programming with C++ version 98, I'm far from a fan of object-oriented programming. Back then, my code was mostly imperative and procedural. Same goes with Python, the second language I learned. Things started changing when I was doing my bachelor's in applied mathematics and my dislike for MatLab, which made me search for classes that were using anything but MatLab, which meant Haskell and Lisp. Lisp had a very interesting approach for a mathematician, as it used the same computational logic as my graphic HP calculator, so I would always aim for classes that were in Lisp (or, for some weird reason, Pascal).


Fast forward over a decade, and everyone that has personally talked to me about programming knows I am passionate about functional programming, and more so, I have experience making non-fp code bases become functional, as well as writing a lot of content about functional programming, especially in Rust, Java, C# and Clojure. So the passion is evident, but **what are the benefits of FP?**


* <inline class="text-orange-500"><b>Referential Transparency</b></inline>, with the use of pure functions your objects don't necessary own data and mutate their own state, you start having Instances/Namespaces/Objects that act on data passed to them, helping testability and providing a safer and more predictable way to understand the state of your data.


* <inline class="text-orange-500"><b>Immutability</b></inline>. At one point, I read in a Clojure book about the fact that if you have mutability, you know nothing about the state of your application, and so, you cannot make any predictions about its behaviour. This is not the case when you deal with immutable code, at any moment in time, when you take a snapshot of your data, you can clearly predict its next state based on `x, y, z` functions.


* <inline class="text-orange-500"><b>Composition and higher order functions</b></inline> are another key aspect and has a lot to do with a simpler way to deal with functions that contain side effects. By using composition, we can always guarantee that our code is predictable, a very interesting feature for testing. Let's use the general example of random data. I have a function that needs to define the initial direction something is moving, but that direction cannot be always the same, so we need random data. However, it makes it very hard to test, as we now have lost prediction of results. However, by passing a function that handles randomize data, we mock that function in test cases to return a desired value. 


* <inline class="text-orange-500"><b>Concurrency</b></inline>. Off-course concurrency is not a functional programming restricted topic, but the previous attributes that we discussed, make it so much clearer on how to deal with it.



> A few books that I can recommend for learning how to become functional thinking or how expand your fp insights are:

* Clojure Applied by Ben Vandgrift and Alex Miller
* Domain Modeling Made Functional by Scott Wlaschin
* Adopting Elixir by Ben Marx, Bruce Tate and José Valim
* Becoming Functional by Joshua Backfield
* Functional Thinking by Neal Ford


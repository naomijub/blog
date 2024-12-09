---
title: Tech Debt
date: 2024-12-09
description: What is tech debt and how to handle it
keywords: "Tech Debt Recognition, Tech Debt"
---

# Tech Debt

Between _"Do it quick"_ and _"Do it right"_, I will always go with _"Do it right"_, which doesn't mean over engineering. It means we should avoid deliberate technical debt, as [Martin Fowler](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html) defines in the tech debt quadrants between reckless/prudent and deliberate/inadvertent (figure 1). This is different from inadvertent technical debt, which usually means a tech debt that we realized we had later. 



<img src="/images/blog_images/tech_debt_quadrant.png" alt="Tech debt quadrants: Reckless and Prudent vs. Deliberate Inadvertent">
Figure 1 - Tech debt quadrants


Tests, in fact untested code, should also be considered a tech debt. I have, multiple times, in my career decided to deliver a "working software" to showcase the product before a **well tested working software**. However, once the proof of concept evolved, my team paid this recklessness cost later, and this was a larger cost than just delivering tested code from the start. So, my position is that **untested code is deliberate reckless tech debt** as we should have planned the test cases beforehand and implement the code as we test it. 


Untested code is very common in game development, where people like to prototype untested games and evolve from there without adding tests. Also, people usually avoid at all costs automated test for gameplay, which 99% of the time is the cause of recurrent bugs. At some point, when people decide to add tests, it is usually just too late or too hard, meaning you a set of useless tests.


> **In short**: Tech Debt is like a bad loan, the quicker we pay it, the less we suffer from it.

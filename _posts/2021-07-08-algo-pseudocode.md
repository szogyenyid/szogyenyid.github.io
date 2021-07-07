---
layout: post
title: "Introduction to Programming: Algorithms & Pseudocodes"
date: 2021-07-07 07:00:00 +0200
category: Learning # Learning | Data Science | Security | Meta | Stories
tags:
    - programming
    - algorithm
    - pseudocode
description: TBW
# last_modified_at: 2021-07-07 09:40:00 +0200
author: Daniel Szogyenyi
---

## Specification to algorithm

In programming, we usually get a specification, which is the description of a problem. In the majority of the times it is something [**declarative**][decimp-decla], and programmers have the task to create an **imperative** solution, such a method that generates the wanted output using the given input.  
So, specification is all about "what to solve" and an algorithm is about "how to solve". The programmer's responsibility is converting the "what" to "how", and the rest of this "Introduction to Programming" course will talk about essentials you must know to be able to do this conversion.

## How to go from "what" to "how"?

If I could give an answer to this question in a tiny paragraph (or even in a tome), I would be a billionaire. No explicit answer exists, and that's why programmers earn money. This is a complex mental activity which is hard to master, always with some room to get better. If we look at programming from different angles, we can see merely different things:

- Science: yeah, just like the name **computer science** suggests, it's science. Algorithms can be handled by mathematical stuff, their correctness can be proven, their complexity can be calculated, and so on.  
- Engineering: an expression similar to "programmer" is "**software engineer**". Some aspects of creating a program is the same as designing a car engine. You have to keep in mind a lot of things in order to prevent components interfering with each other, you have to control them in a way that they could work simultaneosly. There are a lot of written ans unwritten rules to follow.  
- Art: creating an algorithm requires unique ideas, a great programmer should see things that others don't. One of the most famous books about algorithms is titled [The Art of Computer Programming][taocp], I think that means something.

However, we encounter a ton of problems which share similarities or even the exact same ones. We don't have to reinvent the wheel each time we want to use it: there are well-tried solutions worth using again. A good programmer can tell more methods for sorting a list, and they know which method is the fastest in various situations. Keep in mind, when programming, you will have to use learnt algorithms to suit your needs.

## Getting into pseudocodes

> **pseudo-** <span>/sjuː.dəʊ-/ • prefix</span>
> 1. pretended and not real[^fn-pseudo-dictionary]

A pseudocode looks like a piece of code, but no interpreter or compiler on your computer will be able to run it. It's written for your brain in a human language to better understand the logic behind and algorithm.

Let's see a pseudocode for getting the pass ratio of an exam.  
<!--{% highlight %}
Numbers to remember: numberOfStudents, currentStudent, numberOfPasses
Set numberOfStudents to 30
Set currentStudent to 1
Set numberOfPasses to 0
While currentStudent is less than or equal to numberOfStudents:
    If the score of (currentStudent)th student is greater than 60
        Add 1 to numberOfPasses
    Add 1 to currentStudent
Print: (100*numberOfPasses/numberOfStudents) % of students has passed this exam
{% endhighlight %}-->

_Note that everything in this code is a simple, easy-to-do step, like equality check and addition._

As you have seen, a pseudocode contains:  
- The numbers to remember, they are called **variables**
- The **commands** to do, like division and addition
- Commands and decision points that determine the order of the steps: **control flow**

You have just read the description of an algorithm.

## Algorithm

**An algorithm is a method for solving the task.**

- List of valid commands
- Step-by-step
- Finite number of steps
- The next step is always definite
- Only needs a finite amount of memory


[^fn-sample_footnote]: [Cambridge Dictionary][cambridge-pseudo]

[decimp-decla]: https://szogyenyid.github.io/learning/2021/07/07/decla-vs-imper.html#declarative-approach
[taocp]: https://en.wikipedia.org/wiki/The_Art_of_Computer_Programming
[cambridge-pseudo]: https://dictionary.cambridge.org/dictionary/english/pseudo(https://dictionary.cambridge.org/dictionary/english/pseudo)
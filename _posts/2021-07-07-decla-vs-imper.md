---
layout: post
title: "Introduction to Programming: Declarative vs Imperative"
date: 2021-07-07 13:57:00 +0200
categories: # Learning | Data Science | Security | Meta | Stories 
    - learning
tags:
    - declarative
    - imperative
    - programming
description: What is the difference between declarative and imperative paradigms? What do we have to say to the computer, and what kind of questions do we want to answer? I am giving a short summary of the declarative and imperative approaches.
last_modified_at: 2021-07-11 12:04:00 +0200
author: Daniel Szogyenyi
mathjax: true
math: true
---

## Methods of giving a solution

There are several ways of giving a solution for a given problem. Now I would like to introduce two groups of solutions (or even knowledge): **Declarative** and **Imperative**.

### Declarative approach

Declarative knowledge answers the question "What is true?" and makes statements. Using our declarative knowledge, we can always check if a statement we have is true or false (or we don't know).

Example: the square root of a number.  
Our declarative knowledge would say "the square root of a number $$ x $$ is a number $$ y $$ such that $$ y^2 = x $$ and $$ y\geqslant0 $$ ".

So, if someone gives us an $$ x $$ and a $$ y $$, we can tell if $$ y^2=x $$ is true or false.

There are declarative programming languages, where we know the attributes of a good solution, but the computer makes up the way of giving us the answers. SQL is a perfect example, where we only command the computer to "give me the e-mail addresses of all the users, who are signed up for the newsletters, and interested in imperative languages", and the computer _magically_ does this.

### Imperative approach

Imperative knowledge is about giving methods for solving a problem, answering the "How to do it?" question. Using imperative methods, we are able the generate useful data based on another data.

Let's see the square root again.  
A method for determining the square root of $$ x $$ (if $$ x\geqslant0 $$) can be defined as:  

- Guess the square root $$ y $$.
- Improve our guess using this formula:

$$ y = \tfrac{y+\tfrac{x}{y}}{2} $$

- Improve until the guess is _good enough_

You can see, with this method if we get an $$ x $$ we can determine the $$ y $$ for it.

In the imperative programming paradigm we have to write a program, in which we give a step-by-step guide to the computer, so our focus is on the way of the solution. As I would like to teach you the basics of programming, and C is an imperative programming language, you will have to learn this mindset.



<div style="text-align: center;">
    <span style="display:block; float:left;">
        <a href="https://szogyenyid.github.io/learning/2021/07/06/introduction-to-programming-1.html">Previous part: Introduction to Programming #1</a>
    </span>
    &nbsp;
    <span style="display:block; float:right;">
        <a href="https://szogyenyid.github.io/learning/2021/07/08/algo-pseudocode.html">Next part: Algorithms and Pseudocodes</a>
    </span>
</div>
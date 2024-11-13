
---
title: "Algebraic geometry for programmers"
date: "2024-11-13"
---

Algebraic geometry is often considered a scary topic. It smells like an advanced math course with a lot of weird constructions, obscure proofs and no and practical use in sight. However, in this post I will try to convince you, that while modern algebraic geometry can be all of this, there are gems and techniques hidden in this field, which unlucks magic I personally thought was impossible. Chief of these is "automatic theorem proving".

# What is this post about?
This post will teach you just enough about multivariate polynomials and Gröbner bases to be dangerous. It will not contain any proofs, although it will occationally include "intuitive proofs", to help you see why a certain theorem is true. I will try to give you the tools to enable you to ask your computer any geometric question, and have it answer your question automatically. If that sounds as cool to you as it does to me, then this blog post is probably for you.

This post will be covered by examples and code to make it as accessible as possible, but some mathematical machinery is required. If you know what "linear function" means you should be good. I urge you to not be scared by the formulas: they are merely computer code with a 2-dimensional syntax for an unusual language.

# Introductions: what the hell is a multivariate polynomial?
You probably know of polynomial functions such as $f(t) = \frac{1}{2}at^2 + v_0 t + x_0$. This particular function calculates the distance a moving object has travelled at time $t$ given an initial distance of $x_0$, an initial velocity of $v_0$ and an acceleration of $a$. The important property here is that we a variable, $t$, which we can raise to some power, multiply by constants and add together with other suchpowers of $t$. In the world of algebraic geometry, this is known as a univariate polynomial, a polynomial in one variable.

We can add more variables. $f(x, y) = x^2 + y^2$ is a multivariate polynomial function, which calculates the squared length of the vector $(x, y)$. Now, consider an expression like $(x - 1)^2 + (y - 2)^2$. Is this a polynomial function? If we expand the expression, we get that $(x - 1)^2 + (y - 2)^2 = x^2 - 2x - 4y + 5$, which is indeed a polynomial. What does this function compute? Well, it computes the distance between the point $(x, y)$ and the point $(1, 2)$. 

Generally speaking, if we have two polynomial functions $f(x, y)$ and $g(x, y)$, then $f(x, y) + g(x, y)$ and $f(x, y) \cdot g(x, y)$ are also polynomial functions. We can add and multiply polynomial functions and we still get polynomial functions. This enables us to describe a wide variety of computations using polynomials.

## The link between polynomials and geometry


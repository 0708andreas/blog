
---
title: "Algebraic geometry for programmers"
date: "2024-11-13"
---

Algebraic geometry is often considered a scary topic. It smells like an advanced math course with a lot of weird constructions, obscure proofs and no and practical use in sight. However, in this post I will try to convince you, that while modern algebraic geometry can be all of this, there are gems and techniques hidden in this field, which unlucks magic I personally thought was impossible. Chief of these is "automatic theorem proving".

# What is this post about?
This post will teach you just enough about multivariate polynomials and Gröbner bases to be dangerous. It will not contain any proofs, although it will occationally include "intuitive proofs", to help you see why a certain theorem is true. I will try to give you the tools to enable you to ask your computer any geometric question, and have it answer your question automatically. If that sounds as cool to you as it does to me, then this blog post is probably for you.

This post will be covered by examples and code to make it as accessible as possible, but some mathematical machinery is required. If you know what "linear function" means you should be good. I urge you to not be scared by the formulas: they are merely computer code with a 2-dimensional syntax for an unusual language.

# Introductions: what the hell is a multivariate polynomial?
You probably know of polynomial functions such as $f(t) = \frac{1}{2}at^2 + v_0 t + x_0$. This particular function calculates the distance a moving object has travelled at time $t$ given an initial distance of $x_0$, an initial velocity of $v_0$ and an acceleration of $a$. The important property here is that we have a variable, $t$, which we can raise to some power, multiply by constants and add together with other such powers of $t$. In the world of algebraic geometry, this is known as a univariate polynomial, a polynomial in one variable.

We can add more variables. $f(x, y) = x^2 + y^2$ is a multivariate polynomial function, which calculates the squared length of the vector $(x, y)$ (remember Pythagoras formula $a^2 + b^2 = c^2$? This is that). Now, consider an expression like $(x - 1)^2 + (y - 2)^2$. Is this a polynomial function? If we expand the expression, we get that $(x - 1)^2 + (y - 2)^2 = x^2 - 2x - 4y + 5$, which is indeed a polynomial. What does this function compute? Well, it computes the distance between the point $(x, y)$ and the point $(1, 2)$. This is again due to Pythagoras. The distance from $x$ to 1 one the $x$-axis is $|x - 1|$, which squares to $(x - 1)^2$. Hence $a^2 + b^2 = (x - 1)^2 + (y - 2)^2$.

Generally speaking, if we have two polynomial functions $f(x, y)$ and $g(x, y)$, then $f(x, y) + g(x, y)$ and $f(x, y) \cdot g(x, y)$ are also polynomial functions. We can add and multiply polynomial functions and we still get polynomial functions. This enables us to describe a wide variety of computations using polynomials. This might be a good place to note, that we can have more than two variables. In fact, we can have as many as we'd like.

In Julia, multivariate plynomials are provided by the Nemo package:
```julia
julia> using Nemo
julia> R, (x, y) = QQ[:x, :y] # Define the variable we'll be using
julia> (x - 1)^2 + (y - 2)^2 # expand the polynomial from above
x^2 - 2*x + y^2 - 4*y + 5
julia> f = (x - 1)^2 + (y - 2)^2
julia> f(2, 3) # we can also evaluate polynomials from Julia
2
julia> evaluate((x - 1)^2 + (y - 2)^2, [2, 3]) # or evaluation can happen via the evaluate function
2
```

## The link between polynomials and geometry
A lot of geometric concepts can be expressed as polynomial expressions. We just saw how the distance between two points can be computed (well, the square of the distance, but that turns out not to be very important). A circle is all the points that are a fixed distance from the centerpoint. For example the circle with radius 3 and center in $(1, 2)$ is all the points that satisfy $(x - 1)^2 + (y - 2)^2 = 3^2$, which is a polynomial equation. If we have another circle, say every point satisfying $(x - 1)^2 + (y - 1)^2 = 2^2$, the question of whether these to circles intersect becomes the question if these two equations have a common solution. We can do this by hand, but we can also use computers to solve it for us. First, we'll normalize the equations to $(x - 1)^2 + (y - 2)^2 - 3^2 = 0$ and $(x - 1)^2 + (y - 1)^2 - 2^2 = 0$. This is standard practice, and it will turn out be super useful later on.

```julia
julia> using HomotopyContinuation
julia> circle1 = (x - 1)^2 + (y - 2)^2 - 3^2
julia> circle2 = (x - 1)^2 + (y - 1)^2 - 2^2
julia> function solve_system(I) # Define a helper function to solve Nemo polynomials using HomotopyContinuation.jl
           vars = Variable.(symbols(parent(I[1])))
           I_ = [sum([Float64(c) * prod(vars .^ e) for (c, e) in zip(Nemo.coefficients(i), exponent_vectors(i))]) for i in I]
           S = System(I_)
           return real_solutions(HomotopyContinuation.solve(S))
       end

julia> solve_system([circle1, circle2])
1-element Vector{Vector{Float64}}:
 [1.0000000005645042, -0.9999999999999994]
```

Since we work in the plane of real numbers, we ask for the real solutions. In this case, Julia tells us that there is a single solution, which means the circles intersect in exactly one point: $(x = 1, y = -1)$ (up to some numerical inaccuracies).

Technical aside: this works when the equations only have finitely many solutions. If we only gave one of the circle equations as input, there would be infinitely many points satisfying the equation. In this case, Julia would complain that the system isn't zero-dimensional. This simply means that there are too many solutions, so naturally Julia can't find all of them. It is a bit unfortunate, that we can't ask for say 10 of them, but that's life.

Alright, so distances and circles can be described using polynomials. What about lines? A line can be given using a slope and a intersection point on the $y$-axis: $y = ax + b$. Given two points $(x_1, y_1)$ and $(x_2, y_2)$, the line between them is given by $(x_2 - x_1)y = (y_2 - y_1)x + y_1 - (y_2 - y_1)x_1$.

A vector is perpendicular to another if their dot-product is zero. The area of a polygon can be computed using the shoelace formula. All of this is polynomial computations. Any matrix product or linear transformation is just a polynomial computation, which gives us rotations and projections.

# Cool, but so what?
This is where we leave the path of the well-known geometry and enter hard-core algebra. First, let's establish a very simple, but extremely important fact: if two polynomials $f(x, y)$ and $g(x, y)$ has a common zero, i.e. a point $(x_0, y_0)$ such that $f(x_0, y_0) = g(x_0, y_0) = 0$, then $f(x_0, y_0) + g(x_0, y_0) = 0$ because zero plus zero is zero. Also, if $h(x_0, y_0)$ is a root of $f(x, y)$ and $g(x, y)$ is any other polynomial, then $f(x_0, y_0) \cdot g(x_0, y_0) = 0$ because zero time anything is still zero.

Now, let's take en example of how this is useful: Consider two points $A = (x_1, y_1)$ and $B = (x_2, y_2)$. These form a triangle whose third vertex is $C = (0, 0)$. Now, assume that the line from $C$ to $A$ is perpendicular to the line from $C to B$.

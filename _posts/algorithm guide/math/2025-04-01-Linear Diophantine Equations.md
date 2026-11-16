---
title: Linear Diophantine Equations
date: 2025-04-04 10:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, divisibility, number theory]
---

# General

A linear Diophantine equation is one that looks like: $ax +by = c$,  where $a, b, c$ are given integers and we want to figure out the integer solutions of $x, y$

1. The degenerate case is when $a=b=0$, where:
    1. If $c=0$, then $x,y$ have infinite solutions
    2. If $c \neq 0$, then $x,y$ have no solutions
2. Through Bézout’s identity, we know that solutions exist for $ax+by = \gcd(a,b)$
    1. So, we can “scale” our answers if $c\%\gcd{a,b} = 0$
    2. If not, there are no solutions.
3. After finding a viable answer, we can create a set of answers following:
    1. $x' = x+\frac{b}{\gcd}k$
    2. $y'=y-\frac{a}{\gcd}k$

# Solution with maximum value of $x+y$

$x'+y' = x+\frac{b}{\gcd}k + y - \frac{a}{\gcd}k = x+y + \frac{b-a}{\gcd}k$

So:

- If $b-a>0$, then find maximal $k$
- If $b-a<0$, then find minimal $k$
- If $b-a = 0$, then all values of $x+y$ are the same

# Finding number of solutions, and the solutions, in a given interval

We will be given two intervals, each for $x$ and $y$: $[min_x, max_x]$ and $[min_y, max_y]$

Before anything, consider the case where $a=0$ or $b=0$. In each, there will only be one solution, so check if said solution is in the range. If so, then since there are infinitely many of the other variable, every single integer in given range is viable.

We first find the minimal and maximal solutions by shifting our original $x$ solutions from Bézout’s identity. This should be trivial and done in $O(1)$ time.

Then, we do the same for $y$, but we find the $x$s that satisfy such $y$.

All we have to do is then find the intersection of two sets of viable $x$s, one dependent on the $x$ range and the other on the $y$ range.

Since the equation $ax+by=c$ is the equivalent of $\frac{a}{g}x + \frac{b}{g}y = \frac{c}{g}$, we can keep $\gcd(\frac{a}{g}, \frac{b}{g})=1$ this way which simplifies formulas.

---
title: Divide and Conquer
date: 2025-07-13 09:00:00 +0900
categories: [algorithm guide, dynamic programming]
tags: [cs, dynamic programming]
---

The concept of divide and conquer should be familiar, where we can perform efficient operations on an array of size $n$ in $O(n\log n)$ because of it’s binary tree-like structure.

We can apply this property for a common case of DP transformations.

1. One dimensional: calculate DP when given specific index $i$
    
    $$
    dp[i] = \min_{j<i}(dp[j] + \text{cost}(j+1, i))
    $$
    
2. Two dimensional: calculate DP when given stage $(i, j)$. The first value of the pair being a stage, and the second a segment.
    
    $$
    dp(i, j) = \min_{0\le k\le j}(dp(i-1,k-1) + \text{cost}(k, j))
    $$
    
    - We refer to $i-1$ to refer to values that are already computed before $dp(i,j)$

We need to make sure it satisfies the monotonicity condition. If so, we can apply divide & conquer. This basically means $opt(i,j) \le opt(i, j+1)$, where $opt$ denotes the optimal $k$ of given range. This condition holds if quadrangle inequality holds.

[Quadrangle Inequality Properties - Codeforces](https://codeforces.com/blog/entry/86306)

Let $a\le  b\le c\le d$ and $\ge$ in the equation mean optimality (as in higher optimality is more optimal).

$$
C(a,c) + C(b,d) \ge C(a,d) + C(b,c)
$$

This is because we use this property for optimization. $opt(i, j') \le opt(i,j)$ if $j' < j$, so we don’t have to consider as many splitting points.

To minimize runtime, we apply the idea that $opt(i, n/4) \le opt(i, n/2) \le opt(i, 3n/4)$. By recursively keeping track of the lower and upper bounds on $opt$, we reach $O(mn\log n)$ runtime.

So, if we say $m = \frac{l+r}{2}$, then we first compute the optimal $k$ for $m$ in a recursive divide and conquer function. Then, we search $(l,m-1)$ and $m+1, r)$ under the premise that their optimal $k$s are each in the ranges $(l_{opt}, k)$ and $(k, r_{opt})$.

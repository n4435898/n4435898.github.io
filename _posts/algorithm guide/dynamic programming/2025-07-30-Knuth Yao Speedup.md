---
title: Knuth-Yao Speedup
date: 2025-07-30 23:00:00 +0900
categories: [algorithm guide, dynamic programming]
tags: [cs, dynamic programming]
---

Knuth-Yao is an optimization for DP transformations like mentioned below, reducing time complexity from $O(x^3)$ to $O(x^2)$:

$$
dp(i,j) = \min_{i\le k < j}[dp(i,k) + dp(k+1,j) + C(i,j)]
$$

A generic problem would be [파일 합치기 2](https://www.acmicpc.net/problem/13974), where finding the DP translation shouldn’t be so hard.

A prerequisite of Knuth-Yao is that you need to satisfy a certain condition when it comes to monotonicity just like divide & conquer DP.

$$
opt(i,j-1)\le opt(i,j) \le opt(i+1, j)

$$

This prerequisite holds when:

- Let $a\le b\le c\le d$
- $C(b,c) \le C(a,d)$
- Quadrangle inequality holds $C(a,c) + C(b,d) \le C(a,d) + C(b,c)$

[Algorithm]

We’ll process DP states in a way that $dp(i,j-1), dp(i+1,j)$ are computed before $dp(i,j)$. Doing so, we also calculate $opt(i,j-1), opt(i+1,j)$. For calculating $opt(i,j)$, we only need to test all $k$ in range $[opt(i,j-1), opt(i+1,j)]$ instead of every number from $i$ to $j-1$. To process $(i,j)$ pairs in this order it is sufficient to use nested for loops which the outer layer traverses from the maximum to minimum one, and the inner layer traverses from $i+1$ to the maximum one.

[Complexity Proof]

Following the structure of our algorithm (refer to implementation):

$$
\sum_{i=1}^N\sum_{j=i+1}^N[opt(i+1,j) - opt(i,j-1)\\
\sum_{i=1}^N\sum_{j=i}^{N-1}[opt(i+1,j+1)-opt(i,j)]
$$

Once we simplify as so, we can notice cancellation Only the positive terms of $j=N-1$ and $i=1$ are left. Simplified:

$$
\sum_{k=1}^N[opt(i,N)-opt(1,k)]
$$

The worst case appears when $opt(i, N) - opt(1,k) = N$. Then the time complexity is $O(N^2)$. Of course, it’s usually slightly better than this.

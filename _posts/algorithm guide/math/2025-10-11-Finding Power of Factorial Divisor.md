---
title: Finding Power of Factorial Divisor
date: 2025-10-11 21:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, combinatorics]
---

Problem statement: You are given two numbers $n, k$. Find the largest power of $k$, being $x$, such that $n!$ is divisible by $k^x$

# Prime $k$

Since the definition of $n! = 1\times2\times3\times...\times(n-1)\times n$

Every $k$th number is divisible by $k$. So, there are $\lfloor \frac{n}{k} \rfloor$

Same goes for every $k^2$th number being divisible by $k^2$. There are $\lfloor \frac{n}{k^2} \rfloor$

In general, for every $i$  each $k^i$th number adds another +1 to the answer. and there are $\lfloor\frac{n}{k^i}\rfloor$ elements

The final answer is $\lfloor\frac{n}{k}\rfloor + \lfloor\frac{n}{k^2}\rfloor + ...+\lfloor\frac{n}{k^i}\rfloor + ...$

This is known as <u>**Legendre’s formula**</u> (le-jawn-der). The sum is finite since only the first $\log_k{n}$ elements are not 0 because of the floor operation. Thus the runtime is $O(\log_k{n})$

```cpp
int fact_pow (int n, int k) {
    int res = 0;
    while (n) {
        n /= k;
        res += n;
    }
    return res;
}
```

And here’s a literal example. Beautiful equation.

```
4!, 2^7

1x2x3x4x5x6x7x8
  1   2   1   3

4+2+1=7
```

# Composite $k$

Problem statement: 

We can factor $k$ into primes, $k = k_1^{p_1} \cdot k_2^{p_2} \cdot ... \cdot k_m^{p_m}$

For each $k_i$ we find the number of times it’s present, say $a_i$, in $n!$ using the algorithm above.

The answer for composite $k$ should be $\min_{i=1...m}\frac{a_i}{p_i}$

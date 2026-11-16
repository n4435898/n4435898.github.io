---
title: Primality Tests
date: 2025-04-10 09:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, number theory]
---

Purpose: To find out if given number $x$ is a prime or not.

The simplest approach is to brute force, and a bit more efficiently to achieve $O(\sqrt{x})$ runtime.

```cpp
bool isPrime(int x) {
    for (int d = 2; d * d <= x; d++) {
        if (x % d == 0)
            return false;
    }
    return x >= 2;
}
```

You can also improve this by checking 2, and then skipping all even numbers d as well.

# Fermat Primality Test

Fermat’s little theorem states that for prime $p$ and a coprime integer $a$, the following holds: $a^{p-1} \equiv 1\mod p$

We can use this as our base for a primality test. We pick an integer $2\le a \le p-2$ and check if the equation holds or not. If it doesn’t hold, we know that $p$ cannot be a prime number. In this case, we call the base $a$ a Fermat witness for the compositeness of $p$.

However the problem is that it’s possible that the equation holds for a composite number as well. Thus our method doesn’t prove primality. We only say $p$ is probably a prime. We call such base $a$ a Fermat liar in this case.

However to improve the accuracy of the test, we run the test with different $a$s multiple times. If even one says it’s not a prime, then it’s not a prime. We pick random $a$s and control our accuracy by the number of tests.

```cpp
bool probablyPrimeFermat(int n, int iter=5) {
    if (n < 4)
        return n == 2 || n == 3;

    for (int i = 0; i < iter; i++) {
        int a = 2 + rand() % (n - 3);
        if (binpower(a, n - 1, n) != 1)
            return false;
    }
    return true;
}
```

However the really bad news is there are some composite numbers where every single coprime $a$ is a Fermat liar. 561 = 3x11x17 is an example, and these are called Carmichael numbers. So we need to be really lucky and choose a non-coprime base $a$ to correctly solve. It’s very fast and Carmichael numbers are rare, so it’s still sometimes used in practice. For example, there are only 646 Carmichael numbers below $10^9$.

# Miller-Rabin Primality Test

The Miller-Rabin test extends the ideas from the Fermat test.

For an odd number $n$, $n-1$ is even and we can factor out all powers of 2.

This allows us to factorize the equation of Fermat’s little theorem.

$$
\begin{array}{rcll}
n-1 & = & 2^s\cdot{d} \\
a^{n-1}\equiv 1\mod n& \Longleftrightarrow & a^{2^sd}-1\equiv 0 & \pmod{n}\\

& \Longleftrightarrow & (a^{2^{s-1}d}+1)(a^{2^{s-1}d}-1)\equiv 0 & \pmod{n}\\

& \Longleftrightarrow & (a^{2^{s-1}d}+1)(a^{2^{s-2}d}+1)(a^{2^{s-2}d}-1) & \pmod{n}\\

&\vdots&\\

& \Longleftrightarrow & (a^{2^{s-1}d}+1)(a^{2^{s-2}d}+1)\cdots(a^d+1)(a^d-1) & \pmod{n}

\end{array}
$$

If $n$ is a prime, then $n$ has to divide one of these factors since the equations above $\equiv 0$ under $\mod n$. 

That is exactly what we’ll be testing in the Miller-Rabin primality test, which is a more stricter version of the statement of the Fermat test.

For base $2\le a \le n-2$, we check if either holds:

- $a^d \equiv 1 \mod n$
- $a^{2^rd}+1 \equiv 0 \mod{n} \Longleftrightarrow a^{2^rd} \equiv -1 \mod{n}$ (for some $0 \le r \le s-1)$

Which should be very clear with the factorization provided above.

If we find a base $a$ that doesn’t satisfy any of the equalities above, then we found a witness for the compositeness of $n$. In this case, we’ve proven that $n$ is not a prime number.

Similar to the Fermat test, it is also possible that there’s a base $a$ that does satisfy the equalities above even when $n$ isn’t a prime number. In this case base $a$ is called a strong liar.

If a base $a$ satisfies one of the equations, $n$ is only strong probable prime. However there are no numbers like the Carmichael numbers where all non-trivial bases lie. In fact it is possible to show that at most $\frac{1}{4}$ of the bases can be strong liars. Basically, each test hast 75% accuracy. 

Here’s an implementation:

```cpp
using u64 = uint64_t;
using u128 = __uint128_t;

u64 binpower(u64 base, u64 e, u64 mod) {
    u64 result = 1;
    base %= mod;
    while (e) {
        if (e & 1)
            result = (u128)result * base % mod;
        base = (u128)base * base % mod;
        e >>= 1;
    }
    return result;
}

bool check_composite(u64 n, u64 a, u64 d, int s) {
    u64 x = binpower(a, d, n);
    if (x == 1 || x == n - 1)
        return false;
    for (int r = 1; r < s; r++) {
        x = (u128)x * x % n;
        if (x == n - 1)
            return false;
    }
    return true;
};

bool MillerRabin(u64 n, int iter=5) { // returns true if n is probably prime, else returns false.
    if (n < 4)
        return n == 2 || n == 3;

    int s = 0;
    u64 d = n - 1;
    while ((d & 1) == 0) {
        d >>= 1;
        s++;
    }

    for (int i = 0; i < iter; i++) {
        int a = 2 + rand() % (n - 3);
        if (check_composite(n, a, d, s))
            return false;
    }
    return true;
}
```

Before Miller-Rabin, you can test additionally if one of the first few prime numbers is a divisor. This can speed up the test a lot since literally 88% of all numbers have a prime factor smaller than 100.

NOTE THAT u64 is VERY IMPORTANT HERE! If you use a signed long 64 like long long, it won’t work for 32 bit inputs.

# Deterministic Version

Miller showed that it is possible to make the algorithm deterministic by only checking all bases $\le O((\ln{n})^2)$, and Bach later gave a concrete bound of $a \le 2\ln({n})^2$

Further research showed that for testing a 32 bit integer, you only need to check the first 4 prime bases: 2, 3, 5, 7. The smallest number that fails this is 3,215,031,751 = 151x751x28351.

For 64 bit integers, you only have to do the first 12 prime bases: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37

```cpp
bool MillerRabin(u64 n) { // returns true if n is prime, else returns false.
    if (n < 2)
        return false;

    int r = 0;
    u64 d = n - 1;
    while ((d & 1) == 0) {
        d >>= 1;
        r++;
    }

    for (int a : {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}) {
        if (n == a)
            return true;
        if (check_composite(n, a, d, r))
            return false;
    }
    return true;
}
```

It’s possible to do the check with only 7 bases: 2, 325, 9375, 28178, 450775, 9780504, 1795265022. However since these are not prime, you need to check additionally if the number you’re checking is equal to any prime divisor of those bases: 2, 3, 5, 13, 19, 73, 193, 407521, 299210837

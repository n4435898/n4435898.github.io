---
title: Chinese Remainder Theorem
date: 2025-05-07 21:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, number theory]
---

$m=\prod_{i=1}^k m_i$ where $m_i$ are pairwise coprime with each other.

We are then given a system of congruences

$$
\left\{
\begin{aligned}
    a &\equiv a_1 \pmod{m_1} \\
    a &\equiv a_2 \pmod{m_2} \\
      &\vdots \\
    a &\equiv a_k \pmod{m_k}
\end{aligned}
\right.
$$

We want to find $a$ that satisfies this system of congruences.

The corollary is that $x \equiv a (\mod{m})$

Is equivalent to the system of equations:

$$
\left\{
\begin{aligned}
    x &\equiv a_1 \pmod{m_1} \\
    x &\equiv a_2 \pmod{m_2} \\
      &\vdots \\
    x &\equiv a_k \pmod{m_k}
\end{aligned}
\right.
$$

Where $a_i = x (\mod{m_i})$

**[Solution for Two Moduli]**

At first the solution seems really confusing and difficult. I had trouble getting a grasp alone.

Consider the following system:

$$
\left\{
\begin{aligned}
    a &\equiv a_1 \pmod{m_1} \\
    a &\equiv a_2 \pmod{m_2}
\end{aligned}
\right.
$$

We want to find a solution for $a (\mod{m_1m_2})$.

Using the Extended Euclidean Algorithm, we can find Bézout coefficients n1, n2 such that: $n_1m_1 + n_2m_2 = 1$, here note $m_1, m_2$ are coprimes.

In fact, $n_1, n_2$ are just the modular inverses.  $n1 \equiv m_1^{-1}(\mod{m_2})$ $n2 \equiv m_2^{-1}(\mod{m_1})$.

$n_1m_1 \equiv 1 \mod{m_1}$ and $n_2m_2 \equiv 1 \mod{m_2}$

With those coefficients we can define:
$a = a_1n_2m_2 + a_2n_1m_1 \mod{m_1m_2}$

Verification is beautiful: by computing $a\mod{m_1}$ and $a\mod{m_2}$

$$
\begin{array}{rcll} a & \equiv & a_1 n_2 m_2 + a_2 n_1 m_1 & \pmod{m_1}\\   & \equiv & a_1 (1 - n_1 m_1) + a_2 n_1 m_1 & \pmod{m_1}\\   & \equiv & a_1 - a_1 n_1 m_1 + a_2 n_1 m_1 & \pmod{m_1}\\   & \equiv & a_1 & \pmod{m_1} \end{array}
$$

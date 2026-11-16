---
title: Knuth-Morris-Pratt (KMP)
date: 2025-06-10 10:00:00 +0900
categories: [algorithm guide, string]
tags: [cs, string]
---

**[Prefix Function Definition]**

The prefix function for a string $s$ is defined as an array, say $\pi$ where $\pi[i]$ is the length of the longest proper prefix of the substring $s[0...i]$ that is also a suffix of this substring. A proper prefix is a prefix that is not equal to the string itself. By definition, $\pi[0] = 0$.

For example: a prefix function of string “abcabcd” is $[0, 0, 1, 2, 3, 0]$.

Mathematically, it can be written as:

$$
\pi[i] = \max_{k=0...i}\{k: s[0...k-1] = s[i-(k-1)...i]\}
$$

I’ll skip the trivial $O(n^3)$ approach and jump straight to optimization.

The first optimization is that $\pi[i+1] \le \pi[i]+1$. The prefix function can only increase by up to 1 when the index is incremented by one.

Proof is trivial. Let’s say that $\pi[i+1] > \pi[i]+1$ . Then we can take the suffix ending in position $i+1$ with length $\pi[i+1]$ and remove the last character from it. This means the removed version has a suffix length of $\pi[i+1]-1$, which is better than $\pi[i]$. Thus, a contradiction.

$$
\underbrace{\overbrace{s_0 ~ s_1}^{\pi[i] = 2} ~ s_2 ~ s_3}_{\pi[i+1] = 4} ~ \dots ~ \underbrace{s_{i-2} ~ \overbrace{s_{i-1} ~ s_{i}}^{\pi[i] = 2} ~ s_{i+1}}_{\pi[i+1] = 4}
$$

This means when moving to the next position, the value of the prefix sum can either increase by 1, stay the same, or decrease by some amount. We can reduce the complexity to $O(n^2)$ because we know that we can only increment by max 1. So, $O(n)$ string comparisons that each cost $O(n)$ result in $O(n^2)$.

The second optimization is entirely removing string comparisons. Instead of comparing every time, we’ll use a DP approach where we use values of the previous steps.

Let us compute the value of the prefix function $\pi$ for $i+1$. If $s[i+1] = s[\pi[i]]$, then we can say with certainty that $\pi[i+1]=\pi[i]+1$. 

$$
\underbrace{\overbrace{s_0 ~ s_1 ~ s_2}^{\pi[i]} ~ \overbrace{s_3}^{s_3 = s_{i+1}}}_{\pi[i+1] = \pi[i] + 1} ~ \dots ~ \underbrace{\overbrace{s_{i-2} ~ s_{i-1} ~ s_{i}}^{\pi[i]} ~ \overbrace{s_{i+1}}^{s_3 = s_{i + 1}}}_{\pi[i+1] = \pi[i] + 1}
$$

However if $s[i+1] \neq s[\pi[i]]$,  then we need to shorten the string until we find a viable case. In order to speed things up, we should move to the longest length $j<\pi[i]$ such that the prefix property in the position $i$ holds: $s[0...j-1] = s[i-j+1... i]$

$$
\overbrace{\underbrace{s_0 ~ s_1}_j ~ s_2 ~ s_3}^{\pi[i]} ~ \dots ~ \overbrace{s_{i-3} ~ s_{i-2} ~ \underbrace{s_{i-1} ~ s_{i}}_j}^{\pi[i]} ~ s_{i+1}
$$

If we find such length $j$, we again only need to compare the characters $s[i+1]$ and $[j]$. If they are equal, we can assign $\pi[i+1] = j+1$. Otherwise, we need to continue to find smaller values of $j$ where the property is held. When $j$ reaches 0, we either assign 1 or 0 depending on $s[i+1] = s[0].$

Now the final step is to figure out how to effectively find the lengths for $j$. Well, $j$ has to hold the property where $s[0...j-1] = s[i-j+1...i]$. So, we want to set the new $j = \pi[j-1]$.

A more intuitive way to understand this is by the realizing the fact that traversing $s[i-j+1...i]$ is merely a repetition. Also, we want to ensure that on the main scale $j$ is usable as a prefix and suffix. Instead of checking if it forms the same prefix and suffix, we immediately lookup the longest proper prefix that is also a suffix. Boom, took me a while.

Thus, runs in $O(n)$. Below is sample code:

```cpp
vector<int> prefix_function(string s) {
    int n = (int)s.length();
    vector<int> pi(n);
    for (int i = 1; i < n; i++) {
        int j = pi[i-1];
        while (j > 0 && s[i] != s[j])
            j = pi[j-1];
        if (s[i] == s[j])
            j++;
        pi[i] = j;
    }
    return pi;
}
```

# KMP: Search for a Substring in a String

KMP works beautifully once you understand what a prefix function is. All you have to do is combine the strings: If you have text $t$ and you want to locate string $s$, you can do it in $O(n+m)$ time if you create a new string that looks like $s+?+t$ and simply run the prefix function. The $?$ is simply any character not included in either $s$ or $t$.

This approach is so much more clear than what I’ve read from geeksforgeeks previously, and I love the concept now.

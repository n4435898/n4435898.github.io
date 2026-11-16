---
title: Manacher's
date: 2025-07-07 17:00:00 +0900
categories: [algorithm guide, string]
tags: [cs, string]
---

Purpose: Finding all sub-palindromes in $O(N)$

In the worst case, a string may have up to $O(n^2)$ palindromic substrings. However, we can still solve linearly by storing information about palindromes in a compact way. For each position $i$ we will find the number of non-empty palindromes centered at this position.

Palindromes with a common center form a contiguous chain, that is if we have a palindrome of length $l$ centered in $i$, we also have palindromes of length $l-2, l-4$ and also centered on $i$. Therefore we collect the information of all palindromic substrings from one main one.

Palindromes of odd and even lengths are accounted for separately as $d_{odd}[i]$ and $d_{even}[i]$. For palindromes of even length we assume that they’re centered in the position $i$ if their two central characters are $s[i]$ and $s[i-1]$, so they’re somewhat right shifted.

In general you can use string hashing for $O(n\log{n})$ and with suffix trees and fast LCA this problem can be solved in $O(n)$. But there’s a much simpler solution by Manacher.

# Manacher’s Algorithm

For fast calculation, we’ll maintain the exclusive borders $(l, r)$ of the rightmost found sub-palindrome, which means the current rightmost sub-palindrome is $s[l+1], s[l+2], \cdots, s[r-1]$. Initially we set $l=0, r=1$ representing an empty string.

To calculate $d_{odd}[i]$ for the next $i$, all previous values in $d_{odd}$ should already be calculated. We do the following:

- If $i$ is outside the current sub-palindrome, meaning $i\ge r$, we’ll increase $d_{odd}[i]$ if the substring $[i-d_{odd}[i]...i+d_{odd}[i]]$ is a palindrome. We can do this by only checking both end characters. When we find a mismatch, we stop. This means we’ve calculated $d_{odd}[i]$ fully. Update $(l,r)$ and continue with the next $i$,
- If $i$ is inside the current sub-palindrome, meaning $i\le r$, we’ll try to extract information from already calculated values. Let’s mirror the position of $i$ in the sub-palindrome so it moves to $j=l+(r-i)$. This moves $i$ to $j$, a place where calculation should already be completed since $j<i$. Then, we can simply assign $d_{odd}[j]$ to $d_{odd}[i]$. However there are exceptions.
- The exception is when the inner palindrome reaches the borders of the outer one, say $j-d_{odd}[j] \le l$ or $i+d_{odd}[j] \ge r$. Because the symmetry outside the outer palindrome is not guaranteed, you can’t simply assign the value from $j$ to $i$. We need to restrict the length of our palindrome by assigning $d_{odd}[i] = r-i$. After this, we’ll run our trivial algorithm again to try and increase $d_{odd}[i]$ while it’s possible.
- Note that when we run again, as in an expansion via the case $i\ge r$, we’re running based on $i$ and not $j$. Below is a nice illustration:

$$
\ldots\ \overbrace{ \underbrace{ s_{l+1}\ \ldots\ s_j\ \ldots\ s_{j+(j-l)-1}\ }_{\text{palindrome}}\ \ldots\ \underbrace{ s_{i-(r-i)+1}\ \ldots\ s_i\ \ldots\ s_{r-1} }_{\text{palindrome}} }^{\text{palindrome}}\ \underbrace{ \ldots \ldots \ldots \ldots \ldots }_{\text{try moving here}}
$$

Complexity analysis time. It runs in $O(n)$. At first it might seem slower, but $l$ and $r$ cannot be decreased. Therefore it runs in $O(n)$ time.

# Implementation

Below is sample code, as in implementation of our algorithm above:

```cpp
vector<int> manacher_odd(string s) {
    int n = s.size();
    s = "$" + s + "^";
    vector<int> p(n + 2);
    int l = 0, r = 1;
    for(int i = 1; i <= n; i++) {
        p[i] = max(0, min(r - i, p[l + (r - i)]));
        while(s[i - p[i]] == s[i + p[i]]) {
            p[i]++;
        }
        if(i + p[i] > r) {
            l = i - p[i], r = i + p[i];
        }
    }
    return vector<int>(begin(p) + 1, end(p) - 1);
}
```

However, implementing Manacher’s for odd and even separately is deemed difficult. Plus, it’s a tedious process. You can throw in additional # characters at ends to make all indices odd.

$$
abcbcba = ?a?b?c?b?c?b?a?
$$

Thus $d[2i] = 2d_{even}[i]+1$ and $d[2i+1] = 2d_{odd}[i]$. This is important to consider when actually using $d[i]$, or decomposing $d[i]$ to $d_{even}, d_{odd}$

Here’s a setup function:

```cpp
vector<int> manacher(string s) {
    string t;
    for(auto c: s) {
        t += string("#") + c;
    }
    auto res = manacher_odd(t + "#");
    return vector<int>(begin(res) + 1, end(res) - 1);
}
```

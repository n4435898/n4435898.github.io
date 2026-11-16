---
title: 
date: 2025-01-03 19:00:00 +0900
categories: [data structure, segment tree]
tags: [cs, segment tree]
---

Purpose: Store array intervals as a tree for quick updates and access.

Memory: $4N$ if size of array is $N$

- Note, it isn’t $2N$ because $N$ is not always a direct power of 2. To add padding, we do $4N$ for simplicity. We can find the nearest power of 2, $M$, and do $2M$. However for simplicity we just set the size to $4N$
- However, there is a way to reduce this to $2N$ using Euler tour traversal a.k.a. pre-order traversal.

# Basics

I’ll skip this part, since I’m familiar myself. If you're unsure, you can check out this resource. There’s a class based approach and array based approach.

[https://cp-algorithms.com/data_structures/segment_tree.html](https://cp-algorithms.com/data_structures/segment_tree.html)

# Euler Tour Traversal

To reduce the amount of memory, you can renumber the vertices of the tree in the order of a Euler tour traversal (Pre-order traversal).

- Here, obviously, the left child index is $v+1$
- Since the left child is responsible for segment $[l, mid]$, there will be a total of
$2*(mid-l)+1$ elements.
- Therefore, the right child index is $v+2*(mid-l)+2$
- Through this, we can reduce memory to $2N$

(to be continued)

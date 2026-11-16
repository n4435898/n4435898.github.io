---
title: Sqrt Decomposition
date: 2025-08-02 19:00:00 +0900
categories: [algorithm guide, query]
tags: [cs, query]
---

# Sqrt Decomposition

Concept: Having a preprocessed array containing segments of the array of approximately $\sqrt{N}$ size. So, if you want to process a range query, you make your answer based off of the bigger segments/blocks, and then manually process the edge blocks where the range is hanging.

If you want to make a modification, you can usually process it in $O(1)$ time by modifying the segment/block the modification is located it. If it’s a sum query, then you can say
$B[i] += new - old$

Therefore, you can answer most queries in $O(\sqrt{N})$ time.

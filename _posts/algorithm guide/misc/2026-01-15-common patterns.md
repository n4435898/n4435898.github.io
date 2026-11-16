---
title: Common patterns in problems?
date: 206-01-15 10:00:00 +0900
categories: [algorithm guide, misc]
tags: [cs]
---

Here are some I common patterns I see appearing in problems that are just nice to test in case you're stuck.

---

Find minimum/maximum answer
- Binary search for answer

XOR problems
- Performing operations on each bit separately, and later merging

Left-Right Unique Trick
- Using indices of left and right instances to determine if an element is unique or not in a given subrange. 3408 - Non-boring sequences
    

Grid tile flipping
- Check how parity of rows and columns may change.

Range DP time complexity
- Note that you can note already computed ranges to reduce time complexity for range DP to $O(n^2)$.

Segments on circles
- Add {s-n, e} and {s, e+n} for segments that cross the start.

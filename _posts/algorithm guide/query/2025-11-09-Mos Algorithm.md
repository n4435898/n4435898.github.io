---
title: Mo's Algorithm
date: 2025-11-09 15:00:00 +0900
categories: [algorithm guide, query]
tags: [cs, query]
---

Concept: Efficient range queries based off sqrt decomposition for very specific problems. It runs in $O((N+Q)\sqrt{N})$ and only for offline queries, but it has it’s advantages.

The problem with sqrt decomposition is that it can’t perform certain queries. For example, finding the mode (number that appears the most) isn’t really feasible since you need to store information about every single number and how many times they appear. This isn’t efficient because the merge step isn’t quick any more, rather it ends up less efficient than the naive approach.

Mo’s algorithm uses a completely different approach for specific queries like this, tracking a single data structure and the only operations are fast.

The idea is to answer queries in a specific order, based off indices. We’ll answer queries with left indices in block 0, then block 1, and so on. Then, among the queries of a block, we’ll answer based off the sorted order of the right indices.

In practice, the sorting looks like this:

```cpp
struct Query {
    int l, r, idx;
    bool operator<(Query other) const
    {
        return make_pair(l / block_size, r) <
               make_pair(other.l / block_size, other.r);
    }
};
```

Then to process these queries in order, we’ll simply add or remove elements on each ends of the previous query to make it the new query.

This only works for offline queries since we need to sort the queries.

# Implementation

```cpp
void remove(idx);  // TODO: remove value at idx from data structure
void add(idx);     // TODO: add value at idx from data structure
int get_answer();  // TODO: extract the current answer of the data structure

int block_size;

struct Query {
    int l, r, idx;
    bool operator<(Query other) const
    {
        return make_pair(l / block_size, r) <
               make_pair(other.l / block_size, other.r);
    }
};

vector<int> mo_s_algorithm(vector<Query> queries) {
    vector<int> answers(queries.size());
    sort(queries.begin(), queries.end());

    // TODO: initialize data structure

    int cur_l = 0;
    int cur_r = -1;
    // invariant: data structure will always reflect the range [cur_l, cur_r]
    for (Query q : queries) {
        while (cur_l > q.l) {
            cur_l--;
            add(cur_l);
        }
        while (cur_r < q.r) {
            cur_r++;
            add(cur_r);
        }
        while (cur_l < q.l) {
            remove(cur_l);
            cur_l++;
        }
        while (cur_r > q.r) {
            remove(cur_r);
            cur_r--;
        }
        answers[q.idx] = get_answer();
    }
    return answers;
}
```

# Time Complexity Analysis

Sorting takes $O(Q\log{Q})$

We call adding and removing operations $N$ time per block. So, if the block size is $S$, we call $O(\frac{N}{S}N)$ time, or approximately $O(N\sqrt{N})$ times.

The value of cur_l can change up to $O(S)$ times during queries. So an additional $O(SQ)$, or $O(Q\sqrt{N})$.

In total, the time complexity is $O((N+Q)F\sqrt{N})$, $O(F)$ being the complexity of each operation.

# Advanced

For a general optimization, you can precompute your block size instead of doing that during runtime.

To optimize sorting slightly, you can do:

```cpp
bool cmp(pair<int, int> p, pair<int, int> q) {
    if (p.first / BLOCK_SIZE != q.first / BLOCK_SIZE)
        return p < q;
    return (p.first / BLOCK_SIZE & 1) ? (p.second < q.second) : (p.second > q.second);
}
```

You can optimize Mo’s sorting algorithm further using the Hilbert curve:

https://codeforces.com/blog/entry/61203

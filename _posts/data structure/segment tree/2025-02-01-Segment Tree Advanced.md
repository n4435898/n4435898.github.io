---
title: Segment Tree Advanced
date: 2025-02-01 19:00:00 +0900
categories: [data structure, segment tree]
tags: [cs, segment tree]
---

Concept: Using lazy propagation for efficient segment tree operations.

General tip: Focus on how you’ll make a combine function when merging two segments.

# Complex Queries

1. Finding two related elements (e.g. maximum and the number it appears)

You can use pairs and a combine function to pass data.

```cpp
pair<int, int> t[4*MAXN];

pair<int, int> combine(pair<int, int> a, pair<int, int> b) {
    if (a.first > b.first) 
        return a;
    if (b.first > a.first)
        return b;
    return make_pair(a.first, a.second + b.second);
}

void build(int a[], int v, int tl, int tr) {
    if (tl == tr) {
        t[v] = make_pair(a[tl], 1);
    } else {
        int tm = (tl + tr) / 2;
        build(a, v*2, tl, tm);
        build(a, v*2+1, tm+1, tr);
        t[v] = combine(t[v*2], t[v*2+1]);
    }
}

pair<int, int> get_max(int v, int tl, int tr, int l, int r) {
    if (l > r)
        return make_pair(-INF, 0);
    if (l == tl && r == tr)
        return t[v];
    int tm = (tl + tr) / 2;
    return combine(get_max(v*2, tl, tm, l, min(r, tm)), 
                   get_max(v*2+1, tm+1, tr, max(l, tm+1), r));
}

void update(int v, int tl, int tr, int pos, int new_val) {
    if (tl == tr) {
        t[v] = make_pair(new_val, 1);
    } else {
        int tm = (tl + tr) / 2;
        if (pos <= tm)
            update(v*2, tl, tm, pos, new_val);
        else
            update(v*2+1, tm+1, tr, pos, new_val);
        t[v] = combine(t[v*2], t[v*2+1]);
    }
}
```

1. GCD/LCM computation

You can use the exact same approach as standard sum/max problems since you can compute GCD/LCM of two numbers in $O(1)$

1. Counting number of $\Tau$s, searching for the $k$th $\Tau$

Store the number of $\Tau$s in the segment tree to solve the first part. To find the $k$th $\Tau$, you an traverse the segment tree from the root and see if the left side has more or less than $k$ $T$s. You progress for the remainder until you reach the $k$th $T$.

1. Searching for an array prefix with the given amount

What this means is to find the smallest $i$ so that the first $i$ values of the array are $\ge k$ 

1. Finding the first element bigger than given $k$ in given range.

You can use a similar strategy of number 3 and 4.

```cpp
int get_first(int v, int tl, int tr, int l, int r, int x) {
    if(tl > r || tr < l) return -1;
    if(t[v] <= x) return -1;

    if (tl== tr) return tl;

    int tm = tl + (tr-tl)/2;
    int left = get_first(2*v, tl, tm, l, r, x);
    if(left != -1) return left;
    return get_first(2*v+1, tm+1, tr, l ,r, x);
}
```

1. Finding subsegments with the maximal sum

The problem gives a range, and we have to find the subsegment among the range with the maximal sum. Negative value elements are allowed.

We will store 4 values for each vertex: Sum, maximum prefix sum, maximum suffix sum, maximal subsegment. ⇒ The answer is already precomputed.

Here’s how the combination function works:

- Left child, if the optimal subsegment is entirely placed in the segment of the left child.
- Right child, if the optimal subsegment is entirely placed in the segment of the right child.
- The sum of the maximum suffix sum of left child and the maximum prefix sum of the right child, if the optimal subsegment hangs along both left and right children.

```cpp
struct data {
    int sum, pref, suff, ans;
};

data combine(data l, data r) {
    data res;
    res.sum = l.sum + r.sum;
    res.pref = max(l.pref, l.sum + r.pref);
    res.suff = max(r.suff, r.sum + l.suff);
    res.ans = max(max(l.ans, r.ans), l.suff + r.pref);
    return res;
}
```

The code should do the talking. Also, you can make segment tree operations following the standard principles.

To query, jut use the combine function we made previously while digging from root to leaves.

```cpp
data query(int v, int tl, int tr, int l, int r) {
    if (l > r) 
        return make_data(0);
    if (l == tl && r == tr) 
        return t[v];
    int tm = (tl + tr) / 2;
    return combine(query(v*2, tl, tm, l, min(r, tm)), 
                   query(v*2+1, tm+1, tr, max(l, tm+1), r));
}
```

# Saving the Entire Subarrays in each Vertex

You can store a vector or a multilist per node. You might think this requires $O(N^2)$ memory, but it only requires $O(N\log{N})$ memory since each level has a total of $N$ cells.

Here’s a more code-based explanation:

```cpp
vector<int> t[4*MAXN];

void build(int a[], int v, int tl, int tr) {
    if (tl == tr) {
        t[v] = vector<int>(1, a[tl]);
    } else { 
        int tm = (tl + tr) / 2;
        build(a, v*2, tl, tm);
        build(a, v*2+1, tm+1, tr);
        merge(t[v*2].begin(), t[v*2].end(), t[v*2+1].begin(), t[v*2+1].end(),
              back_inserter(t[v]));
    }
}
```

```cpp
int query(int v, int tl, int tr, int l, int r, int x) {
    if (l > r)
        return INF;
    if (l == tl && r == tr) {
        vector<int>::iterator pos = lower_bound(t[v].begin(), t[v].end(), x);
        if (pos != t[v].end())
            return *pos;
        return INF;
    }
    int tm = (tl + tr) / 2;
    return min(query(v*2, tl, tm, l, min(r, tm), x), 
               query(v*2+1, tm+1, tr, max(l, tm+1), r, x));
}
```

```cpp
void update(int v, int tl, int tr, int pos, int new_val) {
    t[v].erase(t[v].find(a[pos]));
    t[v].insert(new_val);
    if (tl != tr) {
        int tm = (tl + tr) / 2;
        if (pos <= tm)
            update(v*2, tl, tm, pos, new_val);
        else
            update(v*2+1, tm+1, tr, pos, new_val);
    } else {
        a[pos] = new_val;
    }
}
```

Also, you can speed this up with fractional cascading. I’ll explain this later, I’m tired right now.

# Lazy Propagation

Lazy propagation is a concept for updating range queries efficiently.

Say you want to add $k$ to $n$ indices. Instead of it running on $O(n\log{N})$, you can make it run in $O(\log{N})$ by only updating until the representative nodes. Later when you want to access a single element, you can add up all nodes on the way. This would be an introductory optimization.

Lazy propagation is slightly more complex, but once understood is trivial. The concept, on a rudimentary level, is just delaying/procrastinating updates for as long as possible. 

Here’s a great introduction to lazy propagation in general:

[https://justicehui.github.io/medium-algorithm/2018/08/24/Seg2/](https://justicehui.github.io/medium-algorithm/2018/08/24/Seg2/)

The reason lazy propagation saves time by such a huge margin is because you’re not updating anything unnecessary. You’re being as minimalistic as possible. Also it doesn’t matter if you need to update later, the time complexity of the next operation that might require further updates is still the same.

Here’s another implementation for setting all values to $k$

```cpp
void push(int v) {
    if (marked[v]) {
        t[v*2] = t[v*2+1] = t[v];
        marked[v*2] = marked[v*2+1] = true;
        marked[v] = false;
    }
}

void update(int v, int tl, int tr, int l, int r, int new_val) {
    if (l > r) 
        return;
    if (l == tl && tr == r) {
        t[v] = new_val;
        marked[v] = true;
    } else {
        push(v);
        int tm = (tl + tr) / 2;
        update(v*2, tl, tm, l, min(r, tm), new_val);
        update(v*2+1, tm+1, tr, max(l, tm+1), r, new_val);
    }
}

int get(int v, int tl, int tr, int pos) {
    if (tl == tr) {
        return t[v];
    }
    push(v);
    int tm = (tl + tr) / 2;
    if (pos <= tm) 
        return get(v*2, tl, tm, pos);
    else
        return get(v*2+1, tm+1, tr, pos);
}
```

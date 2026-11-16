---
title: Persistent Segment Tree
date: 2025-02-07 19:00:00 +0900
categories: [data structure, segment tree]
tags: [cs, segment tree]
---

If we want to remember a previous state for each modification of a segment tree, a persistent segment tree is what we need. We can create this efficiently in terms of both time and memory.

Our goal is to avoid copying the complete tree before each modification, and we don’t want to lose our $O(\log{n})$ advantage.

Note that for any segment tree alteration, we only change $O(\log{n})$ vertices along the path from the root. By storing the segment tree using pointers (a vertex stores pointers of it’s left and right children), then when performing the modification query, we simply need to create new vertices instead of changing the available vertices. Unaffected vertices can still be used by pointing to the old pointers. For a modification query, $O(\log{n})$ new vertices will be created including the new root. So, at most we use up to $O(n\log{n})$ memory.

[Basic point update - CP Algo]

```cpp
struct Vertex {
    Vertex *l, *r;
    int sum;

    Vertex(int val) : l(nullptr), r(nullptr), sum(val) {}
    Vertex(Vertex *l, Vertex *r) : l(l), r(r), sum(0) {
        if (l) sum += l->sum;
        if (r) sum += r->sum;
    }
};

Vertex* build(int a[], int tl, int tr) {
    if (tl == tr)
        return new Vertex(a[tl]);
    int tm = (tl + tr) / 2;
    return new Vertex(build(a, tl, tm), build(a, tm+1, tr));
}

int get_sum(Vertex* v, int tl, int tr, int l, int r) {
    if (l > r)
        return 0;
    if (l == tl && tr == r)
        return v->sum;
    int tm = (tl + tr) / 2;
    return get_sum(v->l, tl, tm, l, min(r, tm))
         + get_sum(v->r, tm+1, tr, max(l, tm+1), r);
}

Vertex* update(Vertex* v, int tl, int tr, int pos, int new_val) {
    if (tl == tr)
        return new Vertex(new_val);
    int tm = (tl + tr) / 2;
    if (pos <= tm)
        return new Vertex(update(v->l, tl, tm, pos, new_val), v->r);
    else
        return new Vertex(v->l, update(v->r, tm+1, tr, pos, new_val));
}
```

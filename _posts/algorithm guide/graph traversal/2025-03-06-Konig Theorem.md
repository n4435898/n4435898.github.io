---
title: Kőnig’s Theorem
categories: [algorithm guide, graph traversal]
tags: [cs, bipartite matching]
---

> *In any bipartite graph, the minimum vertex cover set and the maximum matching set have in fact the same size.*
> 

My simple way of understanding it is to first imagine the set of edges in the maximum matching. Select one vertex of either side for each edge. That means we can make a vertex cover set consisting of N elements, N being the number of edges in the maximum matching.

Now we just have to prove we can’t reduce further vertices from the minimum vertex cover set. This is made obvious by the fact that the edges in the maximum matching don’t share a vertex.

[Read this paper (PDF)](/assets/pdf/cs/konig-theorem-1.pdf){: target="_blank" }

---
title: Bellman-Ford
date: 2025-02-24 19:00:00 +0900
categories: [algorithm guide, graph traversal]
tags: [cs, graph traversal]
---

Purpose: Single source shortest path to all nodes with <u>negative weight edges</u>, modify to deduce solution for problems with negative infinite loops as well.

# Description

Assume that the graph has no negative weight cycle for now.

1. Create an array of distances from the source $d[0 ... n-1]$
2. Let $d[v] = 0, d[v^c] = \infin$, and we’ll fill this array with the answer

The algorithm contains several phases, each phase scanning through the graph and trying to produce **relaxation**:

- Relaxation along edge $(a, b)$ having weight $c$ is an attempt to improve the value $d[b]$ using value $d[a] + c$. So you’re using edge $(a,b)$ and current answer for vertex $a$ to improve the answer for the current vertex.
- For a graph with $n$ vertices, $n-1$ phases are sufficient to correctly calculate the lengths of all shortest paths in the graph. Unreachable vertices will be marked as $d[i] = \infin$

# Implementation

Instead of $n$ lists of edges-edges (like an adjacency matrix), it’s more convenient to have a <u>single list of all edge information</u>. 

Your $Edge$ structure should look something like:

```cpp
struct Edge{
	int u, v, w;
};
```

And here’s a simple implementation, just for reference.

```cpp
void solve(int v){
	vector<int> d(n, INF);
	d[v] = 0;
	
	for(int i=0; i<n-1; i++)
		for(Edge &e : edges)
			if(d[e.u] < INF)
				d[e.v] = min(d[e.v], d[e.u]+e.w);
}
```

This can be improved once by breaking out of our outer for loop earlier instead of looping $n-1$ times ebcause $n-1$ is the worst case. Create a simple boolean value to see if there were any updates in the current phase. If there were none, break out.

Backtracking is simple. You just have to track the current node’s predecessor that is updated when $d[e.u]+e.w < d[e.v]$.

# Proof

I’ll skip this for now since it’s intuitive for me.

# Negative Cycles

Post completion, if you want to check for negative cycles, y<u>ou can run the algorithm once more</u>. If there’s a change, that means there’s a negative cycle in that area. Note that you need to have a lower bound for $d[e]$ since it can be smaller than $-{INF}$.

# Shortest Path Faster Algorithm (SPFA)

This is a modified extention of Bellman-Ford which takes advantage of the fact that not all attempts at relaxation will work.

It looks like a mix of Dijkstra’s and Bellman-Ford. The main idea is to create a queue containing only the vertices that were relaxed but could be further relaxed. Whenever you can relax some neightbor, you add it to the queue.

Time complexity-wise, it is equal to Bellman Ford as in $O(nm)$. In practice, it works even in $O(m)$ on average though due to optimization. Still, there are many counter-examples.

However you must note this algorithm continues forever if there’s a negative cycle. Create a counter that stores how many times a vertex has been relaxed, and stop the algorithm as soon as some vertex got relaxed for the n-th time to prevent infinite loops. Note there is not reason to put a vertex in the queue if it’s already in. Create a separate boolean array like $inq[i]$ to prevent this.

A general algorithm:

```cpp
const int INF = 1000000000;
vector<vector<pair<int, int>>> adj;

bool spfa(int s, vector<int>& d) {
    int n = adj.size();
    d.assign(n, INF);
    vector<int> cnt(n, 0);
    vector<bool> inqueue(n, false);
    queue<int> q;

    d[s] = 0;
    q.push(s);
    inqueue[s] = true;
    while (!q.empty()) {
        int v = q.front();
        q.pop();
        inqueue[v] = false;

        for (auto edge : adj[v]) {
            int to = edge.first;
            int len = edge.second;

            if (d[v] + len < d[to]) {
                d[to] = d[v] + len;
                if (!inqueue[to]) {
                    q.push(to);
                    inqueue[to] = true;
                    cnt[to]++;
                    if (cnt[to] > n)
                        return false;  // negative cycle
                }
            }
        }
    }
    return true;
}
```

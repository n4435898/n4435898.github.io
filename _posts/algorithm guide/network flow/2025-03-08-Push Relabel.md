---
title: Push-Relabel
categories: [algorithm guide, network flow]
tags: [cs, networkk flow]
---

Time complexity: $O(V^2E)$ - It serves as the foundation of other algorithms

```cpp
#include <bits/stdc++.h>
#define ll long long

using namespace std;

const int MAXN = 5000;
const ll INF = 1e15;

int n, m, source, sink;
ll capacity[MAXN + 1][MAXN + 1];
ll flow[MAXN + 1][MAXN + 1];
vector<ll> excess;
vector<ll> height;
vector<ll> next_son;
queue<int> excess_vertexes;

void relabel(int u) {
	ll d = INF;
	for (int v = 1; v <= n; v++) {
		// If the neighbour supports flow.
		if (capacity[u][v] > flow[u][v]) { d = min(d, height[v]); }
	}
	if (d < INF) { height[u] = d + 1; }
}

// Push flow from node u to node v
void push(int u, int v) {
	// delta = How much flow does the edge support
	ll d = min(excess[u], capacity[u][v] - flow[u][v]);
	// Increase the flow from node u to v
	flow[u][v] += d;
	flow[v][u] -= d;
	// Update the excess
	excess[u] -= d;
	excess[v] += d;
	// If node's v excess is equal to delta it means it's the first time.
	if (d && excess[v] == d) { excess_vertexes.push(v); }
}

void discharge(int u) {
	// As long as there is excess flow to push
	while (excess[u] > 0) {
		// The following part can be written as well with a for loop
		if (next_son[u] <= n) {
			int v = next_son[u];
			// If The edge supports more flow and can be pushed towards it
			if (capacity[u][v] > flow[u][v] && height[u] > height[v]) {
				push(u, v);
			} else {
				next_son[u]++;
			}
		} else {
			relabel(u);
			next_son[u] = 1;
		}
	}
}

int main() {
	cin >> n >> m;

	source = 1;
	sink = n;
	excess.resize(n + 1);
	height.resize(n + 1);
	next_son.resize(n + 1);

	height[source] = n;
	excess[source] = INF;

	for (int i = 1; i <= m; i++) {
		int x, y, cap;
		cin >> x >> y >> cap;
		capacity[x][y] += cap;
	}

	for (int i = 1; i <= n; i++) {
		if (i == source) { continue; }
		push(source, i);
	}

	while (!excess_vertexes.empty()) {
		int node = excess_vertexes.front();
		excess_vertexes.pop();
		if (node != source && node != sink) { discharge(node); }
	}

	ll max_flow = 0;
	for (int node = 1; node <= n; node++) { max_flow += flow[node][sink]; }

	cout << max_flow << endl;
}
```

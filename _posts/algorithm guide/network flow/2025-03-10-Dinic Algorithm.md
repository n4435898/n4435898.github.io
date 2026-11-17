---
title: Dinic's Algorithm
categories: [algorithm guide, network flow]
tags: [cs, network flow]
---

[https://cp-algorithms.com/graph/dinic.html](https://cp-algorithms.com/graph/dinic.html)

Here’s an optimized Dinic approach:

```cpp
struct Flow_Edge{
    int u, v;
    int cap;
    Edge(int u, int v, int cap) : u(u), v(v), cap(cap) {}
};

struct Dinic{
    vector<vector<Flow_Edge>> adj;
    vector<int> lv, it;

    Dinic(int n){
        adj.resize(n);
        lv.resize(n);
        it.resize(n);
    }

    void addEdge(int u, int v, long long c){
        adj[u].push_back({v, graph[v].size(), c});
        adj[v].push_back({u, graph[u].size(), 0});
    }

    bool bfs(int s, int t){
        fill(lv.begin(), lv.end(), -1);
        lv[s] = 0;
        queue<int> q;
        q.push(s);

        while(!q.empty()){
            int u = q.front(); q.pop();
            for(auto &e : graph[u]){
                if(lv[e.v] == -1 && e.cap > 0) {
                    lv[e.v] = lv[u] + 1;
                    q.push(e.to);
                }
            }
        }
        return lv[t] != -1;
    }

    long long dfs(int u, int flow, int t){
        if(!flow) return 0;
        if(u == t) return flow;

        for(int &i = it[u]; i < graph[u].size(); i++){
            auto &e = graph[u][i];
            if(lv[e.v] == lv[u] + 1 && e.cap > 0){
                int curFlow = min(flow, e.cap);
                int tempFlow = dfs(e.to, curFlow, t);

                if(tempFlow > 0){
                    e.cap -= tempFlow;
                    graph[e.v][e.u].cap += tempFlow;
                    return tempFlow;
                }
            }
        }
        return 0;
    }

    int maxFlow(int s, int t){
        int total = 0;
        while(bfs(s, t)){
            fill(it.begin(), it.end(), 0);
            while(int flow = sendFlow(s, LLONG_MAX, t)){
                total += flow;
            }
        }
        return total;
    }
};
```

Also, here’s a Sparse table based approach:

```cpp
struct SparseDinic {
    const int flow_inf = 2000000000;
    vector<vector<int>> adj;
    vector<vector<int>> cap;
    vector<vector<int>> flow;

    int n, m = 0;
    int s, t;
    vector<int> level, ptr;
    queue<int> q;

    SparseDinic(int n, int s, int t) : n(n), s(s), t(t) {
        adj.resize(n);
        cap.resize(n, vector<int>(n, 0));
        flow.resize(n, vector<int>(n, 0));

        level.resize(n);
        ptr.resize(n);
    }
	   
		void add_edge(int v, int u, int c){
        adj[v].push_back(u);
        adj[u].push_back(v);
        cap[v][u] = c;
    }

    bool bfs() {
        q.push(s);
        fill(level.begin(), level.end(), -1);
        level[s]=0;
        while (!q.empty()) {
            int v = q.front();
            q.pop();
            for (int u : adj[v]) {
                if (level[u] == -1 && cap[v][u] > flow[v][u]){
                    level[u] = level[v] + 1;
                    q.push(u);
                }
            }
        }
        return level[t] != -1;
    }

    int dfs(int v, int pushed) {
        if (pushed == 0) return 0;
        if (v == t) return pushed;
        for (int &i = ptr[v]; i < adj[v].size(); i++) {
            int u = adj[v][i];
            if (level[u]==level[v]+1 && cap[v][u]-flow[v][u]>0){
                int tr = dfs(u, min(pushed, cap[v][u] - flow[v][u]));
                if(tr>0){
                    flow[v][u] += tr;
                    flow[u][v] -= tr;
                    return tr;
                }
            }
        }
        return 0;
    }

    int max_flow() {
        int f = 0;
        while (bfs()) {
            fill(ptr.begin(), ptr.end(), 0);
            while (int pushed = dfs(s, flow_inf)) {
                f += pushed;
            }
        }
        return f;
    }

    void split(){
        q.push(s);
        fill(level.begin(), level.end(), 0);
        while(!q.empty()){
            int v=q.front();
            q.pop();
            level[v] = 1;

            for (int &u : adj[v]) {
                if(level[u]==0 && cap[v][u] > flow[v][u]){
                    q.push(u);
                }
            }
        }
    }
};
```

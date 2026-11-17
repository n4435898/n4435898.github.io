---
title: LCA in logN
date: 2025-02-09 12:00:00 +0900
categories: [data structure, tree]
tags: [cs, LCA, tree]
---

LCA: Lowest Common Ancestor

We’ll make a 2D array where $par[v][t] = v\text{'s }2^t\text{th ancestor}$. Since $2^{t+1} = 2^t+2^t$, $par[v][t] = par[par[v][t-1]][t-1]$. We’ll use bottom-up DP to fill this array.

Creating a standard parent array should be really easy.

```cpp
void dfs(int curr, int t_depth){
	vt[curr] = true;
	depth[curr] = t_depth;
	for(int &i : g[v]){
		if(!vt[i]){
			par[i][0] = v;
			dfs(i, d+1);
		}
	}
}
```

Once we’re done with that, we can create the full table in linear time.

```cpp
for(int i=1;i<(log2N, say 31);i++){
	for(int j=1;j<=n;j++){
		par[j][i] = par[par[j][i-1]][i-1];
	}
}
```

Really beautiful expansion based on the equation now used as a transformation.

Now using our depth array and parent array, we’ll compute $\text{LCA}(u, v)$

- WLOG, assume $depth[u]\ge depth[v]$. If not, we can simply swap $u,v$
- Let $\text{diff} = depth[u]-depth[v]$. All we need to do is go up by $\text{diff}$ levels from $u$ and then find the LCA using our DP array.
    - However, this would cost $O(n)$ if done naively.
    - But note we can reduce this to $O(\log{n})$ since we have a binary depth array.
        
        ```cpp
        int i = 0;
        while(diff){
        	if(diff&1) u=dp[u][i];
        	diff>=1;
        	i++;
        }
        ```
        
- Now all we need to implement is a trivial LCA when both nodes are at the same level.
    
    ```cpp
    for(int i=(logN, say 31);i>=0;i--){
    	if(dp[u][i] != dp[u][v]){
    		u = dp[u][i];
    		v = dp[v][i];
    	}
    }
    ```
    

Now we can simply query $\text{lca}(a,b)$ in $O(\log{n})$

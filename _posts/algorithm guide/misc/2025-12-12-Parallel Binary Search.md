---
title: Parallel Binary Search
date: 2025-12-12 19:00:00 +0900
categories: [algorithm guide, misc]
tags: [cs, binary search]
---

A naive binary search would cost $O(\log n)$. If we want to initiate $N$ binary searches, this would cost $O(N\log n)$ if done naively.

For a simple binary search where the parameter costs $O(1)$, the time complexity doesn’t change by using parallel binary search. In fact, you still do $O(N\log n)$ operations on default. However, let’s say the time cost of the parameter is $O(T)$. Then, when done naively, it will result in $O(TN\log n)$. With PBS, we only make $2n$ comparisons, thus $O(nT)$ is separated as a cost and $O(N\log n)$ is added separately.

So basically in short, we’re separating the merging process and query process to disperse time complexity. Just refer to my solution for meteors:

8217 - Meteors
While everyone was saying lazy doesn’t work, I solved it with lazy. But the problem is that I had to debug for 2 fucking hours because I didn’t expect long long to be enough. PS, 128 bit runs slower. WOOOOOO

```cpp
#pragma GCC optimize("O3, unroll-loops")

#include<iostream>
#include<vector>
#include<queue>

using namespace std;
typedef long long ll;
typedef pair<int, int> pii;
const ll UPPER_BOUND = 1000000000;

struct Query{
    int l, r, w;
    Query() : l(0), r(0), w(0) {}
    Query(int l, int r, int w) : l(l), r(r), w(w) {}
};

struct Part{
    int l, r;
    vector<int> states;
    Part(int l, int r) : l(l), r(r), states(0) {}
};

vector<ll> segTree;

void propagate(int l, int r, int x){
    if(segTree[x]!=0 && l!=r){
        segTree[x*2+1] = min(segTree[x*2+1] + segTree[x], UPPER_BOUND);
        segTree[x*2+2] = min(segTree[x*2+2] + segTree[x], UPPER_BOUND);
        segTree[x] = 0;
    }
}

void range_add(int tl, int tr, ll v, int l, int r, int x){
    propagate(l, r, x);

    if(tl<=l&&r<=tr){
        segTree[x] = min(segTree[x] + v, UPPER_BOUND);
        propagate(l, r, x);
        return;
    }
    if(l>tr||r<tl) return;

    int m = (l+r)>>1;
    range_add(tl, tr, v, l, m, x*2+1);
    range_add(tl, tr, v, m+1, r, x*2+2);
}

ll point_get(int t, int l, int r, int x){
    propagate(l, r, x);
    if(l==r) return segTree[x];

    int m = (l+r)>>1;
    if(t<=m) return point_get(t, l, m, x*2+1);
    else return point_get(t, m+1, r, x*2+2);
}

void reset(int l, int r, int x){
    segTree[x] = 0;
    if(l==r) return;
    int m = (l+r)>>1;
    reset(l, m, x*2+1);
    reset(m+1, r, x*2+2);
}

int main(){
    cin.tie(0)->sync_with_stdio(0);

    int groupCnt, orbitCnt; cin>>groupCnt>>orbitCnt;
    vector<vector<int>> groups(groupCnt+1, vector<int>());
    vector<ll> req(groupCnt+1);
    vector<int> ans(groupCnt+1);
    segTree.resize(4*orbitCnt+4);
    for(int i=1;i<=orbitCnt;i++){
        int t; cin>>t;
        groups[t].push_back(i);
    }
    for(int i=1;i<=groupCnt;i++) cin>>req[i];

    int queryCnt; cin>>queryCnt;
    vector<Query> queries(queryCnt+1);
    for(int i=1;i<=queryCnt;i++){
        cin>>queries[i].l>>queries[i].r>>queries[i].w;
    }

    queue<Part> parts;
    parts.push(Part(1, queryCnt));
    for(int i=1;i<=groupCnt;i++) parts.front().states.push_back(i);

    while(parts.size()){ //logarithmic
        int last = 0;
        int partCnt = parts.size();

        while(partCnt--){
            Part part = parts.front();
            parts.pop();

            if(part.l==part.r){
                for(int idx : part.states) ans[idx] = part.l;
                continue;
            }

            int m = (part.l+part.r)>>1;
            for(int i=last+1;i<=m;i++){
                if(queries[i].l<=queries[i].r) range_add(queries[i].l, queries[i].r, queries[i].w, 1, orbitCnt, 0);
                else{
                    range_add(1, queries[i].r, queries[i].w, 1, orbitCnt, 0);
                    range_add(queries[i].l, orbitCnt, queries[i].w, 1, orbitCnt, 0);
                }
            }
            last = m;

            Part left(part.l, m), right(m+1, part.r);
            for(int idx : part.states){
                ll tot = 0;
                for(int i : groups[idx]){
                    tot = min(tot + point_get(i, 1, orbitCnt, 0), UPPER_BOUND);
                }
                //cout<<"queryCnt: "<<m<<" idx: "<<idx<<" tot: "<<tot<<"\n";

                if(tot>=req[idx]) left.states.push_back(idx);
                else right.states.push_back(idx);
            }

            parts.push(left); parts.push(right);
        }

        reset(1, orbitCnt, 0);
    }

    for(int i=1;i<=queryCnt;i++){
        if(queries[i].l<=queries[i].r) range_add(queries[i].l, queries[i].r, queries[i].w, 1, orbitCnt, 0);
        else{
            range_add(1, queries[i].r, queries[i].w, 1, orbitCnt, 0);
            range_add(queries[i].l, orbitCnt, queries[i].w, 1, orbitCnt, 0);
        }
    }

    for(int i=1;i<=groupCnt;i++){
        ll tot = 0;
        for(int j : groups[i]){
            tot = min(tot + point_get(j, 1, orbitCnt, 0), UPPER_BOUND);
        }
        if(tot<req[i]) ans[i] = -1;
    }

    for(int i=1;i<=groupCnt;i++){
        if(ans[i]==-1) cout<<"NIE\n";
        else cout<<ans[i]<<"\n";
    }

    return 0;
}

```

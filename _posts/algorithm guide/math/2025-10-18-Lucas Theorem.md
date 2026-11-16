---
title: Lucas Theorem
date: 2025-10-18 19:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, combinatorics]
---

Problem statement: Compute $\binom{n}{k}\mod{p}$

$$
\binom{m}{n} \equiv \prod_{i=0}^{k}\binom{m_i}{n_i} \mod{p}
$$

Where:

- $m = \sum_{i=0}^{k}m_ip^i$
- $n = \sum_{i=0}^{k}n_ip^i$

Essentially, you create a new base-p perspective when p is a prime number. We call those the base-p expansions of $m$ and $n$.

# Combinatorial Proof

(missing :/ haven’t studied yet)

# Implementation

Implementing the concept of Lucas’ Theorem in code is easy, the annoying part is implementing efficient binomial coefficient calculations.

```cpp
#include<iostream>
using namespace std;
typedef long long ll;

int main(){
    ll n, m, mod; cin>>n>>m>>mod;

    ll ans=1;
    while(n>0){
        ll tn, tk;
        tn = n%mod;
        tk = m%mod;
        ans = ans*binomial(tn, tk)%mod;

        n/=mod;
        m/=mod;
    }
    cout<<ans;
    return 0;
}
```

So basically the problem is implementing binomial(). Implementation by Pascal’s is easy. I then thought that standard binomial implementation in $O(N)$ time would cause problems since for a specific problem, max $N$ was 2000, but I overlooked the fact that you can apply modular as you go and keep things in int range.

In summary, here’s a very efficient implementation:

```cpp
#include<iostream>

int nCk(int N, int K, int M){
    int A = 1, B = 1;
    for (int i = N; i > N-K; i--)
        A = A*i%M;
	
    for (int i = K; i >= 2; i--)
        B = B*i%M;

    N = 1, K = M-2;
    while (K){ // modulo inverse implementation
        if (K & 1) N = N*B %M;
        K >>= 1;
        B = B*B %M;
    }
    return A*N %M;
}

int main(){
    long long N, K;
    int M;
    cin>>N>>K>>M;

    int result = 1;
    while (N || K){
        result = result*nCk(N%M, K%M, M) %M;
        N /= M, K /= M;
    }
    cout<<result;
}
```

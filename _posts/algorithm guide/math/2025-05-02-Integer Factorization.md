---
title: Integer Factorization
date: 2025-05-02 22:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, number theory]
---

# Trial Division

The most simple approach, trivial to understand and should be intuitive:

```cpp
vector<long long> trial_division1(long long n) {
    vector<long long> factorization;
    for (long long d = 2; d * d <= n; d++) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

# Wheel Factorization

You can only count the first 2 and then skip all odd numbers. Following the same rule, you can count the first 3 and skip all multiples of 3, and so on for various number. There’s a pattern of $d$ incrementation that appears when you have a certain set of these base primes.

```cpp
vector<long long> trial_division3(long long n) {
    vector<long long> factorization;
    for (int d : {2, 3, 5}) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    static array<int, 8> increments = {4, 2, 4, 2, 4, 6, 2, 6};
    int i = 0;
    for (long long d = 7; d * d <= n; d += increments[i++]) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
        if (i == 8)
            i = 0;
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

A full extension of the wheel method is to check only the prime numbers. Precompute all primes with the Sieve of Eratosthenes and apply these factors:

```cpp
vector<long long> primes;

vector<long long> trial_division4(long long n) {
    vector<long long> factorization;
    for (long long d : primes) {
        if (d * d > n)
            break;
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

Above were all really easy methods of factorization.

# Fermat’s Factorization Method

We can write an odd composite number $n = p\cdot q$ as the difference of two squares:
 $n = (\frac{p+q}{2})^2 - (\frac{p-q}{2})^2$

Fermat’s factorization method guesses the first square, and checks if the remaining part is also a square number. If so, then we hhave found the factors $p, q$.

```cpp
int fermat(int n) {
    int a = ceil(sqrt(n));
    int b2 = a*a - n;
    int b = round(sqrt(b2));
    while (b * b != b2) {
        a = a + 1;
        b2 = a*a - n;
        b = round(sqrt(b2));
    }
    return a - b;
}
```

The algorithm runs in $O(|p-q|)$. However in general it’s pretty slow. By skipping unnecessary values, you can optimize, but it’s not worth.

# Pollard’s $p-1$ Method

It’s very likely that at least one factor of $p$ is $B$-powersmooth for a small $B$.

$B$-powersmooth means that every prime power $d^k$ that divides $p-1$ is at most $B$.

4817191 = 1303 x 3697, and the factors are 31-powersmooth and 16-powersmooth because 1303-1 = 2x3x7x31 and 3697-1 = 16x3x7x11

The idea comes from Fermat’s little theorem. Let $n=p\cdot q$. If $a$ is coprime to $p$, the following holds: $a^{p-1} \equiv 1 \mod{p}$

Extended, $(a^{p-1})^k \equiv a^{k(p-1)} \equiv 1 (\mod{p})$

So for any $M$ with $p-1|M$ we know that $a^M \equiv 1$. Further, $a^M-1 = p\cdot r$ and because of that also $p|\gcd(a^M-1, n)$

If $p-1$ for a factor of $p$ of $n$ divides $M$, we can extract a factor using Euclid’s algorithm.

It is clear that the smallest $M$ that is a multiple of every $B$-powersmooth number is $\operatorname{lcm}(1, 2, \cdots, B)$. Or also:

$$
M = \prod_{\operatorname{prime} q\le B}q^{\lfloor \log_q{B}\rfloor}
$$

```cpp
long long pollards_p_minus_1(long long n) {
    int B = 10;
    long long g = 1;
    while (B <= 1000000 && g < n) {
        long long a = 2 + rand() %  (n - 3);
        g = gcd(a, n);
        if (g > 1)
            return g;

        // compute a^M
        for (int p : primes) {
            if (p >= B)
                continue;
            long long p_power = 1;
            while (p_power * p <= B)
                p_power *= p;
            a = power(a, p_power, n);

            g = gcd(a - 1, n);
            if (g > 1 && g < n)
                return g;
        }
        B *= 2;
    }
    return 1;
}
```

This algorithm is probabilistic and may fail.

# Pollard’s Rho Algorithm

Let the prime factorization of a number be $n = pq$. You follow a pseudo-random sequence $[x_i] =[x_0, f(x_0(, f(f(x_0)), \cdots]$ where $f$ is a polynomial function often in form $f(x) = (x^2+c) \mod n$ 

We are not interested in the sequence itself, but the sequence $\mod p$. Because of the modulo, the sequence will converge into a loop. The birthday paradox suggests that the expected number of elements until a loop is $O(\max(\sqrt{p}, \sqrt[4]n))$

![Desktop View](/assets/img/posts/cs/integer-factorization-1.jpeg){: width="400"}

The algorithm gets its name from the rho($\rho$) shape that the sequence creates.

Now to use this property to our advantage without knowing the number $p$ itself. There is a cycle in the subsequence $i\le j$ if and only if there are two indices $s, t \le j$ such that $x_s \equiv x_t \mod p$

Thus, $x_s-x_t \equiv 0 \mod p$

Therefore $p|\gcd(x_s-x_tl, n)$ since $p$ must be a factor of $n$

If we find two indices $s, t$ with $g = \gcd(x_s-x_t, n) > 1$, we have found a cycle and also a factor $g$ of $n$. There’s a case where $g=n$, which means we haven’t found a proper solution. We must then repeat the algorithm with different parameters such as $x_0$ and constant $c$

**[Floyd’s Cycle-Finding Algorithm]**

Now that we properly understood the general concept of Pollard’s Rho, we can work on implementation. First of all, on the Pollard’s Rho itself, and then further on finding all factors.

We’ll be using Floyd’s cycle-finding algorithm, also known as the Tortoise Hare algorithm which runs in $O(\lambda)$ time where $\lambda$ is $\sqrt[4]n$. Thus it runs in linear time. Here’s an outline of Floyd:

```
function floyd(f, x0):
    tortoise = x0
    hare = f(x0)
    while tortoise != hare:
        tortoise = f(tortoise)
        hare = f(f(hare))
    return true
```

Here’s the full implementation:

```cpp
long long mult(long long a, long long b, long long mod) {
    return (__int128)a * b % mod;
}

long long f(long long x, long long c, long long mod) {
    return (mult(x, x, mod) + c) % mod;
}

long long rho(long long n, long long x0=2, long long c=1) {
    long long x = x0;
    long long y = x0;
    long long g = 1;
    while (g == 1) {
        x = f(x, c, n);
        y = f(y, c, n);
        y = f(y, c, n);
        g = gcd(abs(x - y), n);
    }
    return g;
}
```

For really big numbers, you can use the concept of binary exponentiation as well to prevent overflow.

```cpp
long long mult(long long a, long long b, long long mod) {
    long long result = 0;
    while (b) {
        if (b & 1)
            result = (result + a) % mod;
        a = (a + a) % mod;
        b >>= 1;
    }
    return result;
}
```

Montgomery multiplication implementation is also viable.

# Brent’s Algorithm

Brent’s algorithm is a modification of Floyd’s where instead of incrementing by one and two units you advance by powers of two.

```
function floyd(f, x0):
    tortoise = x0
    hare = f(x0)
    l = 1
    while tortoise != hare:
        tortoise = hare
        repeat l times:
            hare = f(hare)
            if tortoise == hare:
                return true
        l *= 2
    return true
```

Below is a code implementation

```cpp
long long brent(long long n, long long x0=2, long long c=1) {
    long long x = x0;
    long long g = 1;
    long long q = 1;
    long long xs, y;

    int m = 128;
    int l = 1;
    while (g == 1) {
        y = x;
        for (int i = 1; i < l; i++)
            x = f(x, c, n);
        int k = 0;
        while (k < l && g == 1) {
            xs = x;
            for (int i = 0; i < m && i < l - k; i++) {
                x = f(x, c, n);
                q = mult(q, abs(y - x), n);
            }
            g = gcd(q, n);
            k += m;
        }
        l *= 2;
    }
    if (g == n) {
        do {
            xs = f(xs, c, n);
            g = gcd(abs(xs - y), n);
        } while (g == 1);
    }
    return g;
}
```

# Full Factorization with Pollard’s Rho

Below is my code for a full factorization using Pollard’s Rho Algorithm, based on Jinhan’s blog post.

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
typedef __int128 ull;

//primality

ull binpower(ull base, ull e, ull mod){
    ull res = 1;
    base %= mod;
    while(e){
        if(e&1) res = res * base % mod;
        base = base * base % mod;
        e>>=1;
    }
    return res;
}

bool check_composite(ull n, ull a, ull d, ull s){
    ull x = binpower(a, d, n);
    if(x == 1 || x == n-1) return false;
    for(int r=1; r<s; r++){
        x = x * x % n;
        if(x == n-1) return false;
    }
    return true;
};

bool MillerRabin(ull n){
    if(n<4) return n!=1;

    ull r = 0;
    ull d = n-1;
    while((d&1) == 0){
        d>>=1;
        r++;
    }

    for(int s : {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}){
        if(n==s) return true;
        if(check_composite(n, s, d, r)) return false;
    }
    return true;
}

//factorize

ull gcd(ull a, ull b){
   return b ? gcd(b, a%b) : a;
}

ull mult(ull a, ull b, ull mod){
    ull res = 0;
    while(b){
        if(b&1) res = (res + a) % mod;
        a = a<<1 % mod;
        b >>= 1;
    }
    return res;
}

ull f(ull x, ull c, ull mod){
    return (mult(x, x, mod) + c) % mod;
}

ull rho(ull n, ull x0=2, ull c=1){
    ull x=x0, y=x0, g=1;
    while (g==1) {
        x=f(x, c, n);
        y=f(y, c, n);
        y=f(y, c, n);
        g=gcd(abs(x - y), n);
    }
    return g;
}

void factorize(ull n, vector<ull> &factors){
    if(n==1) return;
    if((n&1)==0){
        factors.push_back(2);
        factorize(n>>1, factors);
        return;
    }
    if(MillerRabin(n)){
        factors.push_back(n);
        return;
    }

    ull a, b, c, g = n;

    do{
        if(g==n){
            a = b = rand() % (n - 2) + 2;
            c = rand() % 20 + 1;
        }

        a = f(a, c, n);
        b = f(f(b, c, n), c, n);
        g = gcd(abs(a-b), n);
    } while(g==1);

    factorize(g, factors);
    factorize(n/g, factors);
}

int main(){
    ios_base::sync_with_stdio(false); cin.tie(NULL);
    unsigned long long t; cin>>t;
    ull n = (ull) t;

    vector<ull> factors;
    factorize(n, factors);
    sort(factors.begin(), factors.end());
    for(ull &i : factors){
        cout<<(unsigned long long) i<<"\n";
    }

    return 0;
}

```

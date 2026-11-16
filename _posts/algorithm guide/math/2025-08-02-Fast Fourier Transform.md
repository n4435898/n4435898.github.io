---
title: Fast Fourier Transform
date: 2025-08-02 19:00:00 +0900
categories: [algorithm guide, math]
tags: [cs, math, number theory]
---

Purpose: To multiply two polynomials of length $n$ in $O(n\log{n})$ time, which is better than the trivial multiplication in $O(n^2)$. Same goes for multiplying two long numbers.

Note that it doesn’t work for multiplying arbitrary big polynomials with arbitrary large coefficients or for huge integers. It can easily handle polynomials of size $10^5$ or multiplying two numbers of size $10^6$. Beyond $10^6$ bits, the range and precision will not be enough, though there are more complex variations that can perform better.

# Discrete Fourier Transform (DFT)

Let there be a polynomial of degree $n-1$: $A(x) = a_0x^0 + a_1x^1+\cdots+a_{n-1}x^{n-1}$

WLOG we assume that $n$ is a power of 2. If not, then we simply add the missing terms $a_ix^i$ and set the coefficients to 0.

The theory of complex numbers tells us that the equation $x^n=1$ has $n$ complex solutions, and  the solutions are form $w_{n,k} = e^\frac{2k\pi i}{n}$ with $k = 0 \cdots n-1$. Refer to the complex plane for further proof. 

The extension is that the principal $n$th root $w_n = w_{n,1} = e^\frac{2\pi i}{n}$

The Discrete Fourier Transform of polynomial $A(x)$, given as a vector of the coefficients $a_0, \cdots, a_{n-1}$ is defined as the values of the polynomial at the points $x=w_{n,k}$, thus the vector:

$$
\begin{aligned}    \text{DFT}(a_0, a_1, \dots, a_{n-1}) &= (y_0, y_1, \dots, y_{n-1}) \\    &= (A(w_{n,0}), A(w_{n,1}), \dots, A(w_{n,n-1})) \\    &= (A(w_n^0), A(w_n^1), \dots, A(w_n^{n-1}))\end{aligned}
$$

Note that since $A(x)$ is a function, you can plug these specific $w$s. Basically, you’re transforming the coefficients to $A(og)$.

Inverse Discrete Fourier Transform works similarly:
$\text{InverseDFT}(y_0, y_1, \dots, y_{n-1}) = (a_0, a_1, \dots, a_{n-1})$

Thus, if a direct DFT computes the values of the polynomial at the points at the $n$th roots, the inverse DFT can restore the coefficients of the polynomial using those values.

# DFT Application: Fast Polynomial Multiplication

Let there be two polynomials $A, B$ and we computed the DFTs $\text{DFT}(A), \text{DFT}(B)$.

$$
(A\cdot B)(x) = A(x)\cdot B(x)\\
\text{DFT}(A\cdot B) = \text{DFT}(A)\cdot \text{DFT}(B)\\
A\cdot B = \text{InverseDFT}(\text{DFT}(A)\cdot\text{DFT}(B))
$$

Note this is possible since $\text{DFT}(A), \text{DFT}(B)$ are vectors. That’s why you can multiply them, and the result would be a new vector where each element is the product of each ones from $\text{DFT}(A), \text{DFT}(B)$. This is nothing different from $\text{DFT}(A\cdot B)$.

Let’s think of how we can compute the above. First, we need to compute $\text{DFT}(A)\cdot \text{DFT}(B)$ which we can do in $O(n)$ time by getting the pairwise product of the vector elements. The method for this is trivial.

If we are able to compute the DFT and the InverseDFT in $O(n\log{n})$, then we can compute the product of the two polynomials and long numbers with the same time complexity.

Note the two polynomials have to have the same degree. We can add coefficients with the value 0 to complement for length difference. Also, since the result of the product of the two polynomials has degree $2(n-1)$, we have to double the degrees of each polynomial. From a vector with $n$ values, we cannot reconstruct the desired polynomial with $2n-1$ coefficients (since it’s of degree $2(n-1)$) using InverseDFT.

# Fast Fourier Transform

Fast Fourier Transform is a method that allows computing the DFT in $O(n\log{n})$ time. The basic idea of the FFT is to apply divide and conquer. We divide the coefficient vector of the polynomial into two vectors, recursively compute the DFT for each of them, and combine the results to compute the DFT of the complete polynomial.

Let $A(x)$ be a polynomial with degree $n-1$, where $n$ is a power of 2.

We divide it into two smaller polynomials, one containing only coefficients of even positions and the other containing odd.

$$
A(x) = a_0x^0 + a_1x^1 +\cdots+a_{n-1}x^{n-1}\\
A_0(x) = a_0x^0+a_2x^1 + \cdots + a_{n-2}x^{\frac{n}{2}-1}\\
A_1(x) = a_1x^0+a_3x^1 + \cdots + a_{n-1}x^{\frac{n}{2}-1}

$$

To combine, simply follow $A(x) = A_0(x^2) + xA_1(x^2)$

Here, polynomials $A_0, A_1$ have half as many coefficients as $A$. If we can compute $\text{DFT}(A)$ in linear time using $\text{DFT}(A_0), \text{DFT}(A_1)$, then we get the recurrence $T_{\text{DFT}}(n) = 2T_{\text{DFT}}(\frac{n}{2}) + O(n)$, resulting in $T_{\text{DFT}}(n) = O(n\log{n})$ by the master theorem. Now to try and and accomplish that. 

Our goal:

$$
y_k = \sum_{j=0}^{n-1}a_kw_n^{kj}
$$

Suppose we have computed the vectors in $\text{DFT}(A_0)=(y_k^0)_{k=0}^{n/2-1}$ and $\text{DFT}(A_1)=(y_k^1)_{k=0}^{n/2-1}$. We want to find an expression for $\text{DFT}(A)$. For the firson $\frac{n}{2}$ values, we can use the previously noted equation.

$$
y_k = y_k^0 + w_n^ky_k^1, k=0\cdots \frac{n}{2}-1
$$

This is easier to comprehend if you imagine the actual coefficients moving around to satisfy $A(x) = A_0(x^2) + xA_1(x^2)$.

For the second $\frac{n}{2}$ values we need to find a different expression:

$$
\begin{aligned} y_{k+n/2} &= A\left(w_n^{k+n/2}\right) \\ &= A_0\left(w_n^{2k+n}\right) + w_n^{k + n/2} A_1\left(w_n^{2k+n}\right) \\ &= A_0\left(w_n^{2k} w_n^n\right) + w_n^k w_n^{n/2} A_1\left(w_n^{2k} w_n^n\right) \\ &= A_0\left(w_n^{2k}\right) - w_n^k A_1\left(w_n^{2k}\right) \\ &= y_k^0 - w_n^k y_k^1 \end{aligned}
$$

This should be easy to understand if you read line by line. Just note that $w_n^n=1, w_n^{n/2}=-1$. Trivial to prove by using the polar function of $w_n = e^{2\pi i/n}$. In fact, these are called identities.

Therefore we get:

$$
\begin{aligned} y_k &= y_k^0 + w_n^k y_k^1, &\quad k = 0 \dots \frac{n}{2} - 1, \\ y_{k+n/2} &= y_k^0 - w_n^k y_k^1, &\quad k = 0 \dots \frac{n}{2} - 1. \end{aligned}
$$

Which basically works in linear time.

This pattern a + b, a - b is sometimes called a butterfly. Anyways, we learned how to compute the DFT in $O(n\log {n})$ time.

# Inverse FFT

Let vector ( $y_0, y_1, \cdots, y_{n-1})$ - the values of polynomial $A$ of degree $n-1$ in the points $x=w_n^k$ - be given. We want to restore the coefficients $(a_0, a_1, \cdots, a_{n-1})$ of the polynomial. This problem is known as interpolation, and there are general algorithms for solving it. But in this special case (since we know the values of the points at the roots of unity), we can obtain a much simpler algorithm (that is practically the same as direct FFT).

We can write the DFT, according to definition, in matrix form called the Vandermonde matrix:

$$
\begin{pmatrix} w_n^0 & w_n^0 & w_n^0 & w_n^0 & \cdots & w_n^0 \\ w_n^0 & w_n^1 & w_n^2 & w_n^3 & \cdots & w_n^{n-1} \\ w_n^0 & w_n^2 & w_n^4 & w_n^6 & \cdots & w_n^{2(n-1)} \\ w_n^0 & w_n^3 & w_n^6 & w_n^9 & \cdots & w_n^{3(n-1)} \\ \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\ w_n^0 & w_n^{n-1} & w_n^{2(n-1)} & w_n^{3(n-1)} & \cdots & w_n^{(n-1)(n-1)} \end{pmatrix} \begin{pmatrix} a_0 \\ a_1 \\ a_2 \\ a_3 \\ \vdots \\ a_{n-1} \end{pmatrix} = \begin{pmatrix} y_0 \\ y_1 \\ y_2 \\ y_3 \\ \vdots \\ y_{n-1} \end{pmatrix}
$$

Thus, we can compute the vector $(a_0, a_1, \cdots, a_{n-1})$ by multiplying the vector $(y_0, y_1, \cdots, y_{n-1})$ from the left with the inverse of the matrix:

$$
\begin{pmatrix} a_0 \\ a_1 \\ a_2 \\ a_3 \\ \vdots \\ a_{n-1} \end{pmatrix} = \begin{pmatrix} w_n^0 & w_n^0 & w_n^0 & w_n^0 & \cdots & w_n^0 \\ w_n^0 & w_n^1 & w_n^2 & w_n^3 & \cdots & w_n^{n-1} \\ w_n^0 & w_n^2 & w_n^4 & w_n^6 & \cdots & w_n^{2(n-1)} \\ w_n^0 & w_n^3 & w_n^6 & w_n^9 & \cdots & w_n^{3(n-1)} \\ \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\ w_n^0 & w_n^{n-1} & w_n^{2(n-1)} & w_n^{3(n-1)} & \cdots & w_n^{(n-1)(n-1)} \end{pmatrix}^{-1} \begin{pmatrix} y_0 \\ y_1 \\ y_2 \\ y_3 \\ \vdots \\ y_{n-1} \end{pmatrix}
$$

We simply inversed the Vandermonde matrix, so nothing should be confusing here.

The inverse matrix can be written as:

$$
\frac{1}{n} \begin{pmatrix} w_n^0 & w_n^0 & w_n^0 & w_n^0 & \cdots & w_n^0 \\ w_n^0 & w_n^{-1} & w_n^{-2} & w_n^{-3} & \cdots & w_n^{-(n-1)} \\ w_n^0 & w_n^{-2} & w_n^{-4} & w_n^{-6} & \cdots & w_n^{-2(n-1)} \\ w_n^0 & w_n^{-3} & w_n^{-6} & w_n^{-9} & \cdots & w_n^{-3(n-1)} \\ \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\ w_n^0 & w_n^{-(n-1)} & w_n^{-2(n-1)} & w_n^{-3(n-1)} & \cdots & w_n^{-(n-1)(n-1)} \end{pmatrix}
$$

We can check by seeing that multiplying these leads to the identity matrix.

Thus we obtain:

$$
a_k = \frac{1}{n}\sum_{j=0}^{n-1}y_jw_n^{-kj}
$$

I like to interpret this as: $\vdots = \cdots \times \vdots$ for intuition.

Remember the formula for $y_k:$

$$
y_k = \sum_{j=0}^{n-1}a_kw_n^{kj}
$$

Note that these two show very similar patterns. In fact, $a_k$ can be found by the same divide and conquer algorithm with the direct FFT, only instead of $w_n^k$ we have to use $w_k^{-n}$ and just have to multiply the resulting coefficients by $\frac{1}{n}$.

Really, really beautiful math.

# Implementation

Here’s a recursive implementation of the FFT and the inverse FFT both in one function.

```cpp
using cd = complex<double>;
const double PI = acos(-1);

void fft(vector<cd> & a, bool invert) {
    int n = a.size();
    if (n == 1)
        return;

    vector<cd> a0(n / 2), a1(n / 2);
    for (int i = 0; 2 * i < n; i++) {
        a0[i] = a[2*i];
        a1[i] = a[2*i+1];
    }
    fft(a0, invert);
    fft(a1, invert);

    double ang = 2 * PI / n * (invert ? -1 : 1);
    cd w(1), wn(cos(ang), sin(ang));
    for (int i = 0; 2 * i < n; i++) {
        a[i] = a0[i] + w * a1[i];
        a[i + n/2] = a0[i] - w * a1[i];
        if (invert) {
            a[i] /= 2;
            a[i + n/2] /= 2;
        }
        w *= wn;
    }
}

vector<int> multiply(vector<int> const& a, vector<int> const& b) {
    vector<cd> fa(a.begin(), a.end()), fb(b.begin(), b.end());
    int n = 1;
    while (n < a.size() + b.size()) 
        n <<= 1;
    fa.resize(n);
    fb.resize(n);

    fft(fa, false);
    fft(fb, false);
    for (int i = 0; i < n; i++)
        fa[i] *= fb[i];
    fft(fa, true);

    vector<int> result(n);
    for (int i = 0; i < n; i++)
        result[i] = round(fa[i].real());
    return result;
}
```

You can use the STL complex type to store complex numbers

# Improved Implementation: In-place Computation

To improve efficiency, we’ll use an iterative implementation instead of a recursive one. In the implementation, we created temporary odd and even vectors. However this operation is slow, and if we reorder the elements in a certain way, we don’t have to create these temporary vectors. This means all the calculations can be done in-place inside the main vector itself.

Take a look at the patterns of each division step of our divide & conquer. You first took the odd elements and even elements, then separated. You repeated this process in your new vectors with new indices. But the pattern is, out of the two newly created vectors, one has those with the lowest bit 0 and 1. We can apply this binary concept of division to all steps. For the first step, we check the lowest bit. Then, we check the second lowest bit, and so on.

```
Example for 0~7:
[0 1 10 11 100 101 110 111]
[0 10 100 110] [1 11 101 111]
[[0 100] [10 110]] [[1 101] [11 111]]
```

But there’s a way to do this really efficiently. Since we’re checking the bits in right → left order, we can consider this a sort. You can reverse all bits for each number with padding and then do a simple sort to find the order.

```
[000 001 010 011 100 101 110 111]
[000:0 100:1 010:2 110:3 001:4 101:5 011:6 111:7]
[000:0 001:4 010:2 011:6 100: 1 101:5 110:3 111:7]
[[0 100] [10 110]] [[1 101] [11 111]] 
```

Really, really beautiful.

Now we want to figure out how to combine the two DFTs into one for the complete vector when we have elements in ideal order. We can also perform the union directly inside this vector.

(Side note: My dumbass for a moment thought the $k$ in $y_k^t$ was actual exponentiation and not a notational trick and got confused)

$$
a=\{ [y_0^0, y_1^0, y_2^0, y_3^0], [y_0^1, y_1^1, y_2^1, y_3^1] \}\\
a=\{[y_0^0+w_n^0y_0^1,y_1^0+w_n^1y_1^1,y_2^0+w_n^2y_2^1,y_3^0+w_n^3y_3^1],\\ [y_0^0-w_n^0y_0^1,y_1^0-w_n^1y_1^1,y_2^0-w_n^2y_2^1,y_3^0-w_n^3y_3^1]\}
$$

Thus we have computed the required DFT from the vector $a$. We can apply this concept to all steps of the divide and conquer.

```cpp
using cd = complex<double>;
const double PI = acos(-1);

int reverse(int num, int lg_n) {
    int res = 0;
    for (int i = 0; i < lg_n; i++) {
        if (num & (1 << i))
            res |= 1 << (lg_n - 1 - i);
    }
    return res;
}

void fft(vector<cd> & a, bool invert) {
    int n = a.size();
    int lg_n = 0;
    while ((1 << lg_n) < n)
        lg_n++;

    for (int i = 0; i < n; i++) {
        if (i < reverse(i, lg_n))
            swap(a[i], a[reverse(i, lg_n)]);
    }

    for (int len = 2; len <= n; len <<= 1) {
        double ang = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(ang), sin(ang));
        for (int i = 0; i < n; i += len) {
            cd w(1);
            for (int j = 0; j < len / 2; j++) {
                cd u = a[i+j], v = a[i+j+len/2] * w;
                a[i+j] = u + v;
                a[i+j+len/2] = u - v;
                w *= wlen;
            }
        }
    }

    if (invert) {
        for (cd & x : a)
            x /= n;
    }
}
```

First we apply the bit reversal permutation by swapping each element with the element of the reversed position, then iterate all blocks.

But we can further optimize bit reversal. Since incrementing one leads to all tailing ones from the right being flipped into a zero and the first zero being flipped into a one, this works reverse for reversed binary. Basically, you need to flip all leading ones from the left, and also the next zero.

```cpp
using cd = complex<double>;
const double PI = acos(-1);

void fft(vector<cd> & a, bool invert) {
    int n = a.size();

    for (int i = 1, j = 0; i < n; i++) {
        int bit = n >> 1;
        for (; j & bit; bit >>= 1)
            j ^= bit;
        j ^= bit;

        if (i < j)
            swap(a[i], a[j]);
    }

    for (int len = 2; len <= n; len <<= 1) {
        double ang = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(ang), sin(ang));
        for (int i = 0; i < n; i += len) {
            cd w(1);
            for (int j = 0; j < len / 2; j++) {
                cd u = a[i+j], v = a[i+j+len/2] * w;
                a[i+j] = u + v;
                a[i+j+len/2] = u - v;
                w *= wlen;
            }
        }
    }

    if (invert) {
        for (cd & x : a)
            x /= n;
    }
}
```

You can precompute bit reversal permutation beforehand for even further optimization.

# Number Theoretic Transform

(to be continued)

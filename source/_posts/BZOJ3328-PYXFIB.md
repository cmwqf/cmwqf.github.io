---
title: '[BZOJ3328]PYXFIB'
date: 2020-01-09 11:01:25
tags: [数论]
categories: [BZOJ]
---

# Description

给定$$n,k,p$$且保证$$k|p-1$$，求
$$
\sum_{i=0}^{\lfloor \frac{n}{k}\rfloor}C_n^{i*k}Fib_{i*k}
$$
其中$$Fib$$指斐波那契数列，$$n\le 1e18,k\le 2e5,p\le 1e9$$。

<!--more-->

# Solution

又一次被打败了。。。

看这个式子形式就很不好搞，我们换一种
$$
\sum_{i=0}^n[k|i]C_n^iFib_i
$$
一般看到组合数和某个数相乘之和，首先想到二项式定理，考虑把$$Fib_i$$化成$$x^i$$的形式。

注意到

$$
Fib_i=(A^i)_{1,1}
$$

，其中
$$
A=\left[
\begin{matrix}
1 & 1\\
1 & 0
\end{matrix}
\right]
$$
这样，将二项式定理推广到矩阵形式，我们有
$$
\sum_{i=0}^nC_n^iFib_i=(A+I)^n_{1,1},
I=
\left[
\begin{matrix}
1 & 0\\
0 & 1
\end{matrix}
\right]
$$
然而，那个$$[k|i]$$有什么用呢？

如果你看了上面那篇**原根及其应用**，你就会明白，
$$
[k|i]=\frac{1}{k}\sum_{j=0}^{k-1}w_k^{ij},w_k=g^{\frac{p-1}{k}}
$$
这样，原式等于
$$
\frac{1}{k}\sum_{i=0}^n\sum_{j=0}^{k-1}w_k^{ij}C_n^iFib_i\\
=\frac{1}{k}\sum_{j=0}^{k-1}\sum_{i=0}^{n}w_k^{ij}C_n^iFib_i\\
=\frac{1}{k}\sum_{j=0}^{k-1}(A*w_k^{j}+I)^n_{1,1}
$$
然后直接矩阵快速幂即可。

复杂度$$O(klogn)$$。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;
 
#define LL long long
 
int T;
LL mod, n, k;
 
struct matrix
{
    LL a[2][2];
 
    matrix() { memset(a, 0, sizeof(a)); }
 
    matrix operator * (matrix rhs)
    {
        matrix ans;
        for(int i = 0; i < 2; i++)
            for(int j = 0; j < 2; j++)
                for(int k = 0; k < 2; k++)
                    ans.a[i][j] = (ans.a[i][j] + a[i][k] * rhs.a[k][j] % mod) % mod;
        return ans;
    }
 
};
 
inline LL mpow(LL a, LL x)
{
    LL ans = 1;
    while(x)
    {
        if(x & 1) ans = ans * a % mod;
        a = a * a % mod;
        x >>= 1;
    }
    return ans;
}
 
inline bool check(LL g)
{
    LL x = mod - 1;
    for(LL i = 2; i * i <= x; i++)
        if(x % i == 0)
        {
            if(mpow(g, (mod - 1) / i) == 1) return false;
            while(x % i == 0) x /= i;
        }
    if(x != 1 && mpow(g, (mod - 1) / x) == 1) return false;
    return true;
}
 
inline LL solve(LL w, LL x)
{
    matrix a;
    a.a[0][0] = w + 1, a.a[0][1] = w;
    a.a[1][0] = w, a.a[1][1] = 1;
    matrix ans;
    for(int i = 0; i < 2; i++) ans.a[i][i] = 1;
    while(x)
    {
        if(x & 1) ans = ans * a;
        a = a * a;
        x >>= 1;
    }
    return ans.a[0][0];
}
 
int main()
{
    scanf("%d", &T);
    while(T --)
    {
        scanf("%lld %lld %lld", &n, &k, &mod);
        LL ans = 0, g = 2;
        while( !check(g) ) g ++;
        LL w = 1, wn = mpow(g, (mod - 1) / k);
        for(int i = 0; i < k; i++, w = w * wn % mod) ans = (ans + solve(w, n)) % mod;
        printf("%lld\n", ans * mpow(k, mod - 2) % mod);
    }
    return 0;
}
```


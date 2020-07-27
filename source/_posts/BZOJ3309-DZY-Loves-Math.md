---
title: '[BZOJ3309]DZY Loves Math'
date: 2019-03-08 16:55:45
tags: [数论]
categories: [BZOJ]
---

# Description

求
$$
\sum_{i=1}^n\sum_{j=1}^mf(gcd(i,j))\\
其中f(n)为n质因子分解后的最高项次数
$$
<!--more-->

# Solution

我们设$$n<=m$$，然后又开始大力推式子了：
$$
\sum_{i=1}^n\sum_{j=1}^mf(gcd(i,j))\\
=\sum_{d=1}^nf(d)*\sum_{i=1}^n\sum_{j=1}^m[gcd(i,j)=d]\\
=\sum_{d=1}^nf(d)*\sum_{i=1}^{n/d}\sum_{j=1}^{m/d}[gcd(i,j)=1]\\
=\sum_{d=1}^nf(d)*\sum_{i=1}^{n/d}\sum_{j=1}^{m/d}\sum_{p|gcd(i,j)}\mu(p)\\
=\sum_{d=1}^nf(d)*\sum_{p=1}^{n/d}\mu(p)*\lfloor\frac{n}{dp}\rfloor*\lfloor\frac{m}{dp}\rfloor\\
=\sum_{d=1}^nf(d)*\sum_{p=1,d|p}^{n}\mu(\frac{p}{d})*\lfloor\frac{n}{p}\rfloor*\lfloor\frac{m}{p}\rfloor\\
=\sum_{p=1}^n\lfloor\frac{n}{p}\rfloor*\lfloor\frac{m}{p}\rfloor*\sum_{d|p}f(d)*\mu(\frac{p}{d})
$$
我们考虑怎么求
$$
g(n)=\sum_{d|n}f(d)*\mu(\frac{n}{d})
$$
我们设
$$
n=p1^{a1}*p2^{a2}*...*pk^{ak}\\
d=p1^{b1}*p2^{b2}*...*pk^{bk}
$$
根据$$\mu$$的性质，如果$$ai-bi>=2$$，那么$$\mu$$就是0。

所以，只有$$ai=bi$$或$$ai=bi+1$$。

这样，我们假设$$max(ai)=l$$，那么$$f(d)=l或l-1$$。

那么$$\frac{n}{d}$$就是这k个质数选或者不选的问题了。

我们有一下两个结论：

1.当$$ai\neq aj(i\neq j)$$时，$$g(n)=0$$。

我们考虑如果选了$$t个=l(0<t<k)$$的，那么，如果我们一共选了s个质数，若s为奇数，记为s1，那么答案就会加上$$l\times (-1)\times C(k-t,s1-t)$$，若s为偶数，记为s2，则答案会加上$$l\times 1\times C(k-t,s2-t)$$，而我们考虑到，$$C(k,i)$$中奇数项和等于偶数项和，所以
$$
\sum_{s1=2i-1}C(k-t,s1-t)=\sum_{s2=2i}C(k-t,s2-t)
$$
所以最终的结果加起来等于$$l\times (-1)\times C(k-t,s1-t)+l\times 1\times C(k-t,s2-t)=0$$，同理，当乘上的数是$$l-1$$时，$$g(n)=0$$，所以结论成立。

因此，只有当$$a1=a2=...=an$$时，才会产生贡献。

2.当$$a1=a2=...=ak$$时，$$g(n)=(-1)^{k+1}$$。

我们设$$a1=a2=...=ak=rule$$。

同样的，如果不考虑前面的f，选奇数个和选偶数个的答案$$\mu$$的和都是0，但是我们考虑当这k个数都选的时候，前面的$$f(d)=rule-1$$，而其他的$$f$$都是$$rule$$，所以在加的时候，相当于全部选的时候，$$(rule-1)\times (-1)^k=rule\times (-1)^k+(-1)^{k+1}$$，其中$$rule\times (-1)^k$$可以抵消掉，会多出来一个$$(-1)^{k+1}$$。

所以，$$g(n)=(-1)^{k+1}$$。

------

然后，我们考虑怎么递推求g。

我们考虑线性筛的原理，一个数只会被自己最小的质因子筛掉。

在筛的时候有两种情况：

1. i % prime[j] != 0
     这个时候，$$prime[j]$$和$$i$$互质，且$$prime[j]$$是$$i\times prime[j]$$的最小的质因子（线性筛的原理），
     因此，我们用一数组$$a[i]$$表示i中最小的质因子出现的次数，$$b[i]$$表示i中最小质因子所构成的数值。
     这两个都很好转移。
     然后，此时的$$g[i\times prime[j]]$$就看i是否有平方因子，如果有，那么$$g[i \times prime[j]]$$就是0，否则就是$$g[i]$$的相反数。
     即

```c++
if(i % prime[j])
{
   a[i * prime[j]] = 1; b[i * prime[j]] = prime[j];
   g[i * prime[j]] = a[i] == 1 ? -g[i] : 0;
}
```

2. i % prime[j] == 0

   这个时候，$$i$$里面有$$prime[j]$$的因子，且$$prime[j]$$必定是$$i$$中最小的因子，这个时候，我们可以算出$$i\times  prime[j]$$里面$$prime[j]$$出现的次数和$$prime[j]$$所构成的数值，我们怎么判断$$g[i \times prime[j]]$$的值呢？我们只要看$$i$$中除去$$prime[j]$$这个因子构成的数后的数，记为$$num$$，看$$a[num]$$是否等于$$a[i\times prime[j]]$$，如果相等的话，直接$$g[i\times prime[j]]=-g[num]$$，如果$$g[num]$$等于0，那么$$g[i\times prime[j]]$$也是0，否则，就直接是其相反数。

   特别地，如果当前这个数$$n$$仅仅是由一个质因子的幂次方组成的，那么这个数的$$g[n]=1$$（自己推一下就好了）。

   还有一个要注意的，就是当$$num$$变成了1，而1的所有属性都没有计算，所以要额外考虑，即此时的$$i\times prime[j]$$是$$prime[j]^2$$的形式，那么，$$g[i\times prime[j]]=1$$就好了。

# Code

```c++
#include <cstdio>
#include <iostream>
using namespace std;
 
#define LL long long
 
const int maxN = 1e7 + 100;
 
int n, m, t;
int g[maxN + 1], prime[maxN + 1], tot;
int a[maxN + 1], b[maxN + 1];
bool flag[maxN + 1];
 
inline int read()
{
    int num = 0, f = 1;
    char ch = getchar();
    while(!isdigit(ch)) { if(ch == '-') f = -1; ch = getchar(); }
    while(isdigit(ch)) num = (num << 3) + (num << 1) + (ch ^ 48), ch = getchar();
    return num * f;
}
 
inline void init()
{
    g[1] = 0;
    for(int i = 2; i < maxN; i++)
    {
        if(!flag[i]) prime[++ tot] = i, g[i] = a[i] = 1, b[i] = i;
        for(int j = 1; j <= tot && i * prime[j] < maxN; j++)
        {
            flag[i * prime[j]] = true;
            if(i % prime[j])
            {
                a[i * prime[j]] = 1, b[i * prime[j]] = prime[j];
                g[i * prime[j]] = (a[i] == 1) ? -g[i] : 0;
            }
            else
            {
                a[i * prime[j]] = a[i] + 1, b[i * prime[j]] = b[i] * prime[j];
                int num = i / b[i];
                if(num == 1) g[i * prime[j]] = 1;
                else g[i * prime[j]] = a[num] == a[i * prime[j]] ? -g[num] : 0;
                break;
            }
        }
    }
    for(int i = 1; i < maxN; i++) g[i] += g[i - 1];
}
 
int main()
{
    init();
    t = read();
    while(t --)
    {
        n = read(), m = read();
        if(n > m) swap(n, m);
        int i = 1;
        LL ans = 0;
        while(i <= n)
        {
            int k = min(n / (n / i), m / (m / i));
            ans += 1ll * (g[k] - g[i - 1]) * (n / i) * (m / i);
            i = k + 1;
        }
        printf("%lld\n", ans);
    }
    return 0;
}
```


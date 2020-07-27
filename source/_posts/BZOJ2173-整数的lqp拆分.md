---
title: '[BZOJ2173]整数的lqp拆分'
date: 2019-07-27 17:40:01
tags: [组合数学,生成函数]
categories: [BZOJ]
---

# Description

一个数$$N=a_1+a_2+...+a_m(m>0)$$，它的贡献是
$$
Fib{a_1}*Fib{a_2}*...*Fib_{a_m}
$$
求它的所有整数划分的贡献之和对$$1e9+7$$取模。

<!--more-->

# Solution

设$$F(x)$$是$$Fibonacci$$数列的生成函数，则答案就是
$$
ans=\sum_{i=1}^{\infty}[x^n]F^i(x)=[x^n]\frac{F(x)}{1-F(x)}
$$
我们先推一下$$Fibonacci$$数列（除去第$$0$$项）的生成函数。
$$
F(x)=x+2x^2+3x^3+5x^4+...\\
xF(x)=x^2+2x^3+3x^4+5x^5+...\\
(1-x)F(x)=x+x^2+x^3+2x^4+3x^5+...\\
(1-x)F(x)=x+x^2F(x)\\
F(x)=\frac{x}{1-x-x^2}
$$
则
$$
ans=[x^n]\frac{x}{1-2x-x^2}
$$
这个，我们该怎么搞呢？

~~查看题解后~~，我们考虑用递推，我们设$$G(x)=\frac{x}{1-2x-x^2}$$，$$g(n)$$是第$$x^n$$前的系数，则
$$
G(x)-2xG(x)-x^2G(x)=x\\
G(x)=2xG(x)+x^2G(x)+x
$$
然后我们考虑第$$n$$项的系数，$$G(x)$$的第$$n$$项显然就是$$g(n)$$，然后$$2xG(x)$$第$$n$$项的系数就是$$2*g(n-1)$$，$$x^2G(x)$$第$$n$$项的系数就是$$g(n-2)$$，所以
$$
g(n)=2*g(n-1)+g(n-2)
$$
因为$$G(x)$$没有常数项，所以通过上面那个$$G(x)=2xG(x)+x^2G(x)+x$$，我们可以知道$$g(1)=1,g(2)=2$$。

然后大力递推就好了。

真是妙哉，妙哉！！

# Code

```c++
#include <cstdio>
using namespace std;

#define int long long

const int maxN = 1e6 + 100, mod = 1e9 + 7;

int f[maxN + 1], n;

#undef int
int main()
#define int long long
{
    scanf("%lld", &n);
    f[1] = 1, f[2] = 2;
    for(int i = 3; i <= n; i++) f[i] = (2 * f[i - 1] + f[i - 2]) % mod;
    printf("%lld", f[n]);
    return 0;
} 
```


---
title: DZY Loves Math
date: 2019-03-08 16:55:45
tags: [数论]
---

[DZY Loves Math](https://www.lydsy.com/JudgeOnline/problem.php?id=3309)

题意：

求
$$
\sum_{i=1}^n\sum_{j=1}^mf(gcd(i,j))\\
其中f(n)为n质因子分解后的最高项次数
$$
<!--more-->

sol:

我们设$n<=m$，然后又开始大力推式子了：
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
\sum_{d|n}f(d)*\mu(\frac{n}{d})
$$

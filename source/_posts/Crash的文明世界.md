---
title: Crash的文明世界
date: 2019-03-02 12:36:45
tags: [dp,组合数学]
---

[Crash的文明世界](https://www.luogu.org/problemnew/show/P4827)

题意：

给定一棵树和k，求：
$$
F(i)=\sum_{j=1}^ndist(i,j)^k
$$
sol:

这一题有一个弱化版，就是$k=1$，是一个非常简单的换根dp(其实我都觉得不是dp):

[STA-Station](https://www.luogu.org/problemnew/show/P3478)

而且，这一题也用了那一题的思路。

废话少说，来看这一题：

先推式子。
$$
\sum_{j=1}^ndist(i,j)^k=\sum_{j=1}^n\sum_{t=0}^{min(dist(i,j),k)}S(k,t)*t!*C(dist(i,j),t)
$$
我们假设$dist(i,j)>k$，毕竟，k的范围要比n要小得多。

则原式变为：
$$
\sum_{j=1}^n\sum_{t=0}^kS(k,t)*t!*C(dist(i,j),t)\\
=\sum_{t=0}^kS(k,t)*t!*\sum_{j=1}^nC(dist(i,j),t)
$$
前面那个式子直接暴力计算，我们主要需要计算的，是
$$
\sum_{j=1}^nC(dist(i,j),t)
$$
首先我们假设1为根，然后设$f[u][k]$为以$u$为根的子树中所有节点与其距离的k次方之和。


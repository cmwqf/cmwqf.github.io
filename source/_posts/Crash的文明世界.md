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
<!--more-->

sol:

这一题有一个弱化版，就是$k=1​$，是一个非常简单的换根dp(其实我都觉得不是dp):

[STA-Station](https://www.luogu.org/problemnew/show/P3478)

而且，这一题也用了那一题的思路。

废话少说，来看这一题：

先推式子。
$$
\sum_{j=1}^ndist(i,j)^k=\sum_{j=1}^n\sum_{t=0}^{min(dist(i,j),k)}S(k,t)*t!*C(dist(i,j),t)
$$
我们假设$dist(i,j)>k​$，毕竟，k的范围要比n要小得多。

则原式变为：
$$
\sum_{j=1}^n\sum_{t=0}^kS(k,t)*t!*C(dist(i,j),t)\\
=\sum_{t=0}^kS(k,t)*t!*\sum_{j=1}^nC(dist(i,j),t)
$$
前面那个式子直接暴力计算，我们主要需要计算的，是
$$
\sum_{j=1}^nC(dist(i,j),t)
$$
首先我们假设1为根，然后设$f[u][t]​$为以$u​$为根的子树中所有节点$v​$的$C(dist(u,v),t)​$之和。

我们再设一个$g[u][t]$指整棵树中除了$u$的子树的所有节点$v$的$C(dist(u,v),t)$之和。

我们用$v\in u​$表示v在u的子树里，则上面的定义可以写成：
$$
f[u][t]=\sum_{v\in u}C(dist(u,v),t)\\
g[u][t]=\sum_{v\notin u}C(dist(u,v),t)
$$
我们知道：
$$
C(n,m)=C(n-1,m)+C(n-1,m-1)
$$
所以，我们就能进行转移啦：
$$
f[u][t]=\sum_{v\in u}C(dist(u,v),t)\\
=[t=0]+\sum_{v\in u,u\neq v}\sum_{w\in v}C(dist(v,w)+1,t)
$$
为什么要加上那一个$[t=0]$，因为如果t=0的话，$0^0=1$，所以要在统计子树的同时统计自己的贡献。

这个可以直接特判掉：
$$
f[u][0]=1+\sum_{v\in u}f[v][0]
$$
剩下的就是$t>1$的情况，就不用特别地写$u\neq v$了：
$$
\sum_{v\in u}\sum_{w\in v}C(dist(v,w)+1,t)\\
=\sum_{v\in u}\sum_{w\in v}(C(dist(v,w),t)+C(dist(v,w),t-1))\\
=\sum_{v\in u}f[v][t]+f[v][t-1]
$$
这就好转移了。

但是，怎么求g呢？
$$
g[u][t]=\sum_{v\notin u}C(dist(u,v),t)\\
=\sum_{v\notin pa}C(dist(pa,v)+1,t)+\sum_{v\in pa}C(dist(pa,v)+1,t)-\sum_{v\in u}C((dist(u,v)+1)+1,t)
$$
其中，最后那个式子中的$dist(u,v)+1$指的是：$dist(pa,v)(v\in pa且v\in u)$

所以，
$$
g[u][t]=g[pa][t]+g[pa][t-1]+f[pa][t]+f[pa][t-1]-f[u][t]-2*f[u][t-1]-f[u][t-2]
$$
然后，然后好像就好了。

第二类斯特林数就暴力计算就好了。

**PS：我的代码风格可能有较大改变，因为学习了大佬 @menci 的代码风格。**

上代码：


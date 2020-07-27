---
title: '[BZOJ2159]Crash的文明世界'
date: 2019-03-02 12:36:45
tags: [dp,组合数学]
categories: [BZOJ]
---

# Description

给定一棵树和k，求：
$$
F(i)=\sum_{j=1}^ndist(i,j)^k
$$
<!--more-->

# Solution

这一题有一个弱化版，就是$$k=1$$，是一个非常简单的换根dp(其实我都觉得不是dp):

[STA-Station](https://www.luogu.org/problemnew/show/P3478)

而且，这一题也用了那一题的思路。

废话少说，来看这一题：

先推式子。
$$
\sum_{j=1}^ndist(i,j)^k=\sum_{j=1}^n\sum_{t=0}^{min(dist(i,j),k)}S(k,t)*t!*C(dist(i,j),t)
$$
我们假设$$dist(i,j)>k$$，毕竟，k的范围要比n要小得多。

则原式变为：
$$
\sum_{j=1}^n\sum_{t=0}^kS(k,t)*t!*C(dist(i,j),t)\\
=\sum_{t=0}^kS(k,t)*t!*\sum_{j=1}^nC(dist(i,j),t)
$$
前面那个式子直接暴力计算，我们主要需要计算的，是
$$
\sum_{j=1}^nC(dist(i,j),t)
$$
首先我们假设1为根，然后设$$f[u][t]$$为以$$u$$为根的子树中所有节点$$v$$的$$C(dist(u,v),t)$$之和。

我们再设一个$$g[u][t]$$指整棵树中除了$$u$$的子树的所有节点$$v$$的$$C(dist(u,v),t)$$之和。

我们用$$v\in u$$表示v在u的子树里，则上面的定义可以写成：
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
为什么要加上那一个$$[t=0]$$，因为如果t=0的话，$$0^0=1$$，所以要在统计子树的同时统计自己的贡献。

这个可以直接特判掉：
$$
f[u][0]=1+\sum_{v\in u}f[v][0]
$$
剩下的就是$$t>1$$的情况，就不用特别地写$$u\neq v$$了：
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
其中，最后那个式子中的$$dist(u,v)+1$$指的是：$$dist(pa,v)(v\in pa且v\in u)$$

所以，
$$
g[u][t]=g[pa][t]+g[pa][t-1]+f[pa][t]+f[pa][t-1]-f[u][t]-2*f[u][t-1]-f[u][t-2]
$$
然后，然后好像就好了。

第二类斯特林数就暴力计算就好了。

注意：在计算g的时候，$$u==1$$的时候无需计算，而且计算要放在循环外面，不然叶子节点就计算不到，我就因为这个一直过不去样例（丢脸）。

**PS：我的代码风格可能有较大改变，因为学习了大佬 @menci 的代码风格。**

# Code

```c++
#include <cstdio>
#include <iostream>
using namespace std;

const int maxN = 1e5 + 100, mod = 10007;

struct Node
{
    int to, next;
}edge[maxN * 2 + 1];

int S[200][200], n, k, fac[200];
int tot, head[maxN + 1];
int f[maxN + 1][200], g[maxN + 1][200];

inline int read()
{
    int num = 0, f = 1;
    char ch = getchar();
    while(!isdigit(ch)) {if(ch == '-') f = -1; ch = getchar(); }
    while(isdigit(ch)) num = (num << 3) + (num << 1) + (ch ^ 48), ch = getchar();
    return num * f;
}

inline void add(int x, int y)
{
    tot ++;
    edge[tot].to = y;
    edge[tot].next = head[x];
    head[x] = tot;
}

inline void dfs1(int u, int pa)
{
    f[u][0]=1;
    for(int i = head[u]; i; i = edge[i].next)
       if(edge[i].to != pa)
       {
           int v = edge[i].to;
           dfs1(v, u);
           f[u][0] = (f[u][0] + f[v][0]) % mod;
           for(int j = 1; j <= k; j++) f[u][j] = (f[u][j] + f[v][j] + f[v][j - 1]) % mod;
       }
}

inline void dfs2(int u, int pa)
{
    if(u != 1)
    {
        g[u][0] = ((g[pa][0] + f[pa][0] - f[u][0]) % mod + mod) % mod;
        g[u][1] = ((g[pa][1] + g[pa][0] + f[pa][1] + f[pa][0] - f[u][1] - 2 * f[u][0]) % mod + mod) % mod;
        for(int j = 2; j <= k; j++)
            g[u][j] = ((g[pa][j] + g[pa][j - 1] + f[pa][j] + f[pa][j - 1] - f[u][j] - 2 * f[u][j - 1] - f[u][j - 2]) % mod + mod) % mod;
    }
    for(int i = head[u]; i; i = edge[i].next)
       if(edge[i].to != pa) dfs2(edge[i].to, u);
}

int main()
{
    n = read(), k = read();
    for(int i = 1; i < n; i++)
    {
        int x = read(), y = read();
        add(x, y); add(y, x);
    }
    fac[0] = 1;
    for(int i = 1; i <= k; i++) fac[i] = fac[i - 1] * i % mod;
    S[0][0] = 1;
    for(int i = 1; i <= k; i++)
       for(int j = 1; j <= i; j++) S[i][j] = (S[i - 1][j - 1] + S[i - 1][j] * j % mod) % mod;
    dfs1(1, 0);
    dfs2(1, 0);
    for(int i = 1; i <= n; i++)
    {
        int ans = 0;
        for(int j = 0; j <= k; j++) 
            ans = (ans + S[k][j] * fac[j] % mod * (f[i][j] + g[i][j]) % mod) % mod;
        printf("%d\n", ans);
    }
    return 0;
}
```


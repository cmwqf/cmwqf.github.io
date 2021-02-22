---
title: CF1307G Cow and Exercise
date: 2021-02-22 15:56:11
tags: [网络流, 线性规划]
categories: [Codeforces]
---

# Description

给定$$n$$个点$$m$$条边的有向图（无重边自环），每条边有边权。

一共$$q$$次询问，每次给定一个数$$x_i$$，我们可以增加某些边的边权，要求在总增加量不超过$$x_i$$的前提下让$$1$$到$$n$$的最短路最大，问这个最大值是多少。

其中$$2\le n\le 50,1\le m\le n(n-1),q\le 10^5$$。

<!--more-->

# 线性规划对偶性

**什么是线性规划？**

我们有$$n$$个变量$$x_1,x_2,\dots,x_n$$，然后有$$m$$条限制，第$$i$$条限制为
$$
\sum_{j=1}^na_{j,i}x_j\le c_i
$$
我们的目标是求
$$
\max \sum_{i=1}^nb_ix_i
$$
上面的$$\le$$可以取反，$$\max$$也可以变成$$\min$$。

这一类问题称为线性规划问题，在$$OI$$中一般采用单纯形进行解决。但是，有许多线性规划问题由于问题本身的特殊性可以用网络流解决。

**那么，线性规划的对偶问题又是什么呢？**

对于上面这个线性规划，其对偶问题为

我们有$$m$$个变量$$y_1,y_2,\dots,y_m$$，有$$n$$条限制，第$$i$$条限制为
$$
\sum_{j=1}^ma_{i,j}y_j\ge b_i
$$
目标是
$$
\min \sum_{i=1}^mc_iy_i
$$
**那么，如何直观地理解对偶问题呢？**

实际上，我们可以从经济学的角度理解对偶问题。

为了方便起见，不妨设原问题为问题$$a$$，其对偶问题为问题$$b$$。

有两个公司$$A,B$$，其中，$$A$$公司是制作产品的，$$B$$公司是回收原料的。

现在，有$$n$$个产品，$$m$$种原料，第$$i$$个产品需要用$$a_{i,j}$$个第$$j$$种原料。$$c_i$$表示第$$i$$种原料有$$c_i$$个，$$b_i$$表示第$$i$$种产品的利润为每个$$b_i$$元。

问题$$a$$是对于$$A$$公司而言的，公司$$A$$要选择每种产品制作多少个，它的目标是在满足要求的情况下总利润最大。设它决定第$$i$$个产品制作$$x_i$$个，那么$$A$$公司制作产品一定要满足原料的个数在限制范围内，即
$$
\sum_{j=1}^na_{j,i}x_j\le c_i
$$
它所希望的总利润最大即
$$
\max \sum_{i=1}^nb_ix_i
$$
而问题$$b$$是针对$$B$$公司而言的，现在$$B$$公司想要从$$A$$公司手中收购原材料，那么显然它要向$$A$$公司报价。设第$$i$$种材料的报价为$$y_i$$。而$$A$$公司也不傻，它同意这个报价当且仅当对于每一种产品，其原材料卖给$$B$$公司的收益不小于产品的利润（否则，不如不卖这些原材料，做成产品，收益更大）。而$$B$$公司的目标自然是最小化购入成本。

那么，$$A$$公司同意的条件为
$$
\sum_{j=1}^m a_{i,j}y_j\ge b_i
$$
而$$B$$公司希望的最小化成本为
$$
\min \sum_{i=1}^mc_iy_i
$$
这样，我们就能很直观地看出对偶问题的关系。可以发现，这两个问题是互为对偶的。

**那么，这两个问题有什么性质？**

这里，我们不加证明地给出线性规划的对偶性：
$$
\max \sum_{i=1}^nb_ix_i=\min \sum_{i=1}^mc_iy_i
$$
感性理解一下也是可以的：如果前者更大，那么$$A$$公司为什么要进行这个交易来减少收益呢？显然不合理。如果后者更大，那么$$A$$公司为什么要制作产品，倒手卖原材料岂不更好？因此，两者只能相等。

但是注意，虽然上述理解过程中我们以产品，材料来便于理解，但实际上$$x_i,y_i,c_i,d_i,a_{i,j}$$不一定是整数，甚至不一定是正数，在实际应用中不要混淆。

线性规划的对偶性让我们可以把$$\max$$转为$$\min$$，也可以把较复杂的问题简化，从而有利于解题。当我们看到一个线性规划问题十分复杂困难时，也许其对偶问题比较简单。

# Solution

这道题看起来十分不可做的样子，但是无法暴力以及较小的数据范围启发我们往线性规划的方面想。

注意到最短路是可以转化为线性规划的形式的。设$$d_u$$为$$1$$到$$u$$的最短路，那么原问题可以形式化地写为：

我们有如下条件
$$
d_u+val_{u,v}+c_{u,v}\ge d_v\\
d_u,c_{u,v}\ge 0
$$
其中$$val_{u,v}$$为边的长度，$$c_{u,v}$$为边增加的长度。

原本还有条件应该是$$\sum c_{u,v}\le x_i$$，但是注意到询问非常多，如果限制与$$x_i$$有关的话应该不太行。这种有那么多询问的题目应该是每次询问在一个特定的候选集合中找符合条件的。

因此我们考虑对于某个答案$$D$$，求出其所需要的最小的$$\sum c_{u,v}$$。

那么限制为
$$
d_u-d_v+c_{u,v}\ge -val_{u,v}\\
d_n-d_1\ge D\\
d_u,c_{u,v}\ge 0\\
$$
我们要求出
$$
\min \sum c_{u,v}
$$
在这个问题中一共有$$n+m$$个变量，分别为$$d_1,\dots,d_n$$以及$$c_{u,v}$$。

考虑这个问题的对偶问题，设$$F$$为上面第二个条件对偶后的变量，

则限制为
$$
\left\{
\begin{array}{lr}
\sum y_{u,v}-\sum y_{v,u}-F\le 0, & u=1\\
\sum y_{u,v}-\sum y_{v,u}\le 0, & \forall u=2,\dots,n-1\\
\sum y_{u,v}-\sum y_{v,u}+F\le 0, & u=n\\
y_{u,v}\le 1, & \forall(u,v)\in E
\end{array}
\right.
$$
而我们的目标是
$$
\max D\times F-\sum y_{u,v}val_{u,v}
$$
观察限制，我们断言，
$$
\forall 1<u<n,\sum y_{u,v}=\sum y_{v,u}\\
u=1,\sum y_{u,v}=\sum y_{v,u}+F\\
u=n,\sum y_{u,v}=\sum y_{v,u}-F
$$
为什么呢？我们把前三个限制组成的不等式全部相加，因为这是一张图，那么不等式左边和不等式右边都是$$0$$，说明不等式是取等的。那么对于每个不等式都是取等的。

观察这个性质，很明显，这个相当于以$$1$$为源点，$$n$$为汇点的网络流，$$y_{u,v}$$为$$(u,v)$$的流量，对于$$1<u<n$$满足流量守恒，而$$F$$是整个网络的流量。每条边的流量上界为$$1$$。

那么对于一个$$D$$，我们只需要检查所有$$F$$与$$y_{u,v}$$的取值中使得$$D\times F-\sum y_{u,v}val_{u,v}$$最大的那一个是否$$\le x_i$$即可。

对于一个固定的$$F$$，后面的$$\sum y_{u,v}val_{u,v}$$最小值显然是以$$val_{u,v}$$为代价的最小费用流，因此我们只需要对于每个$$F$$，求出最小费用流，每次询问的时候逐一检查即可。

但是我们显然不能枚举$$D$$，我们只需要枚举所有二元组$$(F,C)$$，此时$$D\le \frac{x_i+C}{F}$$，对此取$$\min$$即可。

注意到$$F$$只有$$O(n)$$种取值，因此每次检查的时间复杂度为$$O(n)$$，总时间复杂度为$$O(nq)$$。

实际上根据费用流一些结论，这$$n$$个取值构成凸壳，可以在凸壳上二分，但是本题并不需要。

对于每个$$F$$求费用流可以建立超级源汇，往$$1$$和$$n$$连流量为$$F$$的边跑最小费用最大流，但这是不必要的。考虑求费用流的过程，如果每次增广单位流量，那么$$F$$的最小费用流就是$$F-1$$的最小费用流加上这次增广的结果，但是如果采用多路增广这个方法就不行了。

时间复杂度$$O(maxflow+nq)$$。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

#define LL long long

const int maxN = 55;
const LL INF = 1e18;

struct Node
{
	int to, value, cost, next;
}edge[maxN * maxN * 4 + 1];

int n, m, q, cnt, s, t;
int head[maxN + 1], tot = 1, pre[maxN + 1], id[maxN + 1];
LL dis[maxN + 1], ans[maxN + 1];
bool flag[maxN + 1];

inline void add(int x, int y, int t, int c)
{
	edge[++ tot] = (Node){y, t, c, head[x]}; head[x] = tot;
	edge[++ tot] = (Node){x, 0, -c, head[y]}; head[y] = tot;
}

inline bool spfa()
{
	for(int i = 1; i <= t; i++) dis[i] = INF, pre[i] = id[i] = 0, flag[i] = false;
	queue<int> q;
	q.push(s); dis[s] = 0;
	while(!q.empty())
	{
		int u = q.front(); q.pop();
		flag[u] = false;
		for(int i = head[u]; i; i = edge[i].next)
		{
			int v = edge[i].to, w = edge[i].cost;
			if(!edge[i].value || dis[u] + w >= dis[v]) continue;
			dis[v] = dis[u] + w;
			pre[v] = u; id[v] = i;
			if(!flag[v]) flag[v] = true, q.push(v);
		}
	}
	return pre[t] > 0;
}

inline void MCMF()
{
	while( spfa() )
	{
		cnt ++;
		ans[cnt] = ans[cnt - 1] + dis[t];
		for(int now = t; now != s; now = pre[now])
			edge[ id[now] ].value --, edge[ id[now] ^ 1 ].value ++;	
	}
}

int main()
{
	scanf("%d %d", &n, &m);
	for(int i = 1; i <= m; i++)
	{
		int x, y, t;
		scanf("%d %d %d", &x, &y, &t);
		add(x, y, 1, t);
	}

	s = 1, t = n;
	MCMF();

	scanf("%d", &q);
	while(q --)
	{
		int x;
		scanf("%d", &x);
		double res = 1e18;
		for(int i = 1; i <= cnt; i++) res = min(res, (double)(x + ans[i]) / (double)(i));
		printf("%.10lf\n", res);
	}
	return 0;
}
```

# 总结

在以后做题的过程中，如果题目很难下手，可以往线性规划上面想，而如果线性规划的式子可以写成如下形式：

限制为
$$
\left\{
\begin{array}{lr}
d_u-d_v+x_{u,v}\ge w_{u,v}\\
d_u\ge 0, x_{u,v}\ge 0
\end{array}
\right.
$$
目标为
$$
\min \sum c_{u,v}x_{u,v}
$$


那么就可以对偶为费用流问题。
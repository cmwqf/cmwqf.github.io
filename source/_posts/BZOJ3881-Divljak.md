---
title: '[BZOJ3881]Divljak'
date: 2020-03-29 09:53:52
tags: [AC自动机, 字符串]
categories: [BZOJ]
---

# Description

给定一个字符串集合$$S$$，由$$s_1,s_2,..s_n$$组成，且$$\sum|s_i|\le 2e6$$。

由$$q$$个询问，每次询问有两种：

1. 往集合$$T$$中加入一个字符串$$t$$。

2. 询问$$s_x$$是集合$$T$$中多少个字符串的子串。

保证$$\sum |t|\le 2e6,n,q\le 1e5$$。

<!--more-->

# Solution

考虑到有多个字符串，所以构建$$S$$的$$AC$$自动机。

这个时候如果$$s$$是$$t$$的子串，那么$$s$$一定是$$t$$在$$AC$$自动机上经过的某个节点在$$fail$$树上的祖先，如果不考虑重复出现，那么我们只要把每个节点祖先全部加一即可，但是现在有它只统计串的个数，而不是出现次数，所以，我们要转化为数颜色。

对于每一个$$t$$，我们在$$S$$的$$AC$$自动机上跑，把所经过的节点增加一个颜色。

那么，我们的询问就是问$$fail$$树上某个节点子树内的颜色个数。

这种树上数颜色的问题经典做法是离线下来扫描线，然而这道题不能离线，所以我们只能另寻他路。

考虑这道题有什么特殊性质？它每次都会把同一颜色的点全部给你。

那么，我们能不能通过一定的树上差分，使得这些点的祖先只会被统计到一次贡献呢？

实际上是可以的：我们先在每个点上加上$$1$$，然后，我们将这些点按照$$dfs$$序排序，然后我们对排序后相邻两个点的$$LCA$$上$$-1$$，然后求某个点的子树和就可以知道这个点内的颜色个数。

为什么是对的呢？画一下图可以感性理解一下。

因为是按照$$dfs$$序排序，我们询问的是一个子树，所以我们的询问统计到的一定是排序后序列的一个区间。

这个区间内的点和相邻点的$$LCA$$一定在我们询问的子树中，有$$m$$个点，就会有$$m-1$$个$$LCA$$，所以贡献为$$1$$。而其他不在该区间的点及相关的$$LCA$$必然不会在这个子树中（否则矛盾），因此这样做就是对的。

**注意本题求$$LCA$$要用树链剖分，否则会被卡。**

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxN = 2e6 + 10;

struct AC
{
	int fail;
	int trans[26];
}st[maxN + 1];

int n, q, cnt, r;
int p[maxN + 1], t[maxN + 1];
int dfn[maxN + 1], dep[maxN + 1], size[maxN + 1];
int fa[maxN + 1], top[maxN + 1], wson[maxN + 1];
int C[maxN + 1];
char s[maxN + 1];

vector<int> son[maxN + 1];

inline int read()
{
	int num = 0, f = 1;
	char ch = getchar();
	while( !isdigit( ch ) ) { if(ch == '-') f = -1; ch = getchar(); }
	while( isdigit( ch ) ) num = (num << 3) + (num << 1) + (ch ^ 48), ch = getchar();
	return num * f;
}

inline void insert(int id)
{
	int n = strlen(s + 1), now = 0;
	for(int i = 1; i <= n; i++)
	{
		int c = s[i] - 'a';
		if(!st[now].trans[c]) st[now].trans[c] = ++ cnt;
		now = st[now].trans[c];
	}
	p[id] = now;
}

inline void get_fail()
{
	queue<int> q;
	for(int i = 0; i < 26; i++)
		if(st[0].trans[i]) q.push(st[0].trans[i]);
	while(!q.empty())
	{
		int x = q.front(); q.pop();
		for(int i = 0; i < 26; i++)
			if(st[x].trans[i]) st[ st[x].trans[i] ].fail = st[ st[x].fail ].trans[i], q.push(st[x].trans[i]);
			else st[x].trans[i] = st[ st[x].fail ].trans[i];
	}
}

inline void dfs1(int u, int pa)
{
	fa[u] = pa;
	dep[u] = dep[pa] + 1;
	size[u] = 1;
	for(int i = 0; i < son[u].size(); i++)
	{
		int v = son[u][i];
		dfs1(v, u);
		size[u] += size[v];
		if(!wson[u] || size[ wson[u] ] < size[v]) wson[u] = v;
	}
}

inline void dfs2(int u, int t)
{
	dfn[u] = ++ cnt;
	top[u] = t;
	if(wson[u]) dfs2(wson[u], t);
	for(int i = 0; i < son[u].size(); i++)
	{
		int v = son[u][i];
		if(v == wson[u]) continue;
		dfs2(v, v);
	}
}

inline int LCA(int x, int y)
{
	while(top[x] != top[y])
	{
		if(dep[ top[x] ] < dep[ top[y] ]) swap(x, y);
		x = fa[ top[x] ];
	}
	if(dep[x] > dep[y]) swap(x, y);
	return x;
}

inline int lowbit(int x) { return x & -x; }

inline void change(int x, int num)
{
	for(int i = x; i <= cnt; i += lowbit(i)) C[i] += num;
}

inline int query(int x)
{
	int ans = 0;
	for(int i = x; i; i ^= lowbit(i)) ans += C[i];
	return ans;
}

inline bool comp(int a, int b) { return dfn[a] < dfn[b]; }

inline void work()
{
	int n = strlen(s + 1), now = 0;
	t[ r = 1 ] = now;
	for(int i = 1; i <= n; i++)
	{
		int c = s[i] - 'a';
		now = st[now].trans[c];
		t[++ r] = now;
	}
	sort(t + 1, t + r + 1, comp);
	r = unique(t + 1, t + r + 1) - t - 1;
	for(int i = 1; i <= r; i++)
	{
		change(dfn[ t[i] ], 1);
		if(i > 1) change(dfn[ LCA(t[i - 1], t[i]) ], -1);
	}
}

int main()
{
	n = read();
	for(int i = 1; i <= n; i++)
	{
		scanf("%s", s + 1);
		insert(i);
	}

	get_fail();
	
	for(int i = 1; i <= cnt; i++) son[ st[i].fail ].push_back(i);

	cnt = 0;
	dfs1(0, 0);
	dfs2(0, 0);

	q = read();
	while(q --)
	{
		int op = read(), x;
		if(op == 1) scanf("%s", s + 1), work();
		else x = p[ read() ], printf("%d\n", query(dfn[x] + size[x] - 1) - query(dfn[x] - 1));
	}
	return 0;
}
```

# 总结

遇到子树数颜色的问题，如果给了这个颜色的所有点，那么我们可以用按照$$dfs$$序排序的方法，使得这个颜色的点对所有祖先贡献为$$1$$，这个思想非常重要，而且巧妙，一定要加以运用。
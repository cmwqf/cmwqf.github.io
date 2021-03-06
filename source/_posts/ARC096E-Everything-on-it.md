---
title: ARC096E Everything on it
date: 2020-02-28 13:06:07
tags: [组合数学, 容斥原理]
categories: [Atcoder]
---

# Description

有一个面馆，一共有$$n$$种调料，我们要选若干碗面，满足：

1、没有$$2$$种面放的调料的集合相同

2、每种调料至少出现在$$2$$碗面中

求选择的方案数，答案对$$mod$$取模。

其中$$mod$$为质数，$$n\le 3000$$。

<!--more-->

# Solution

直接算很难算，所以我们考虑计算补集。

但是，因为它要求每一个都满足这个条件，补集为存在某一个不满足这个条件，仍然不好直接算。

因此，我们考虑容斥。

我们钦定$$m$$个元素不满足这个条件，即有至少$$m$$个元素出现在$$0/1$$个集合中。

首先，这个选择方案数显然是$$C(n,m)$$。

然后，因为选出来的这些有的出现$$0$$次，有的出现$$1$$次，所以我们把它们分开。

我们枚举$$i$$表示$$m$$中有$$i$$个是出现$$1$$次的，这个选择有$$C(m,i)$$种，我们再枚举分成$$j$$个集合，那么这一部分钦定的方案数就是
$$
C(n,m)\sum_{i=0}^m C(m,i)\sum_{j=1}^i S(i,j)
$$
其中，$$S(n,m)$$是第二类斯特林数。

接下来，我们考虑剩下的$$n-m$$个随便选。

首先，其中一种情况，是只由剩下的元素组成的集合，这个方案数是$$2^{2^{n-m}}$$，因为剩下的$$n-m$$个元素可以组成$$2^{n-m}$$集合，然后每个集合选不选就是$$2^{2^{n-m}}$$。

还有一种情况，就是剩下的元素和我们钦定的一些元素构成一个集合。

考虑我们钦定的元素已经构成了$$j$$个集合，而且每个集合是独立的，我们现在让这$$j$$个集合中的每个集合去选剩下的$$n-m$$个元素有哪些和它组成集合。显然，每个集合的选择方案数是$$2^{n-m}$$，那么总的选择方案数就是$$(2^{n-m})^j=2^{j(n-m)}$$。

所以随便选的方案数就是$$2^{2^{n-m}}·2^{j(n-m)}$$。

那么，总的方案数就是
$$
ans=\sum_{m=0}^n(-1)^mC(n,m)\sum_{i=0}^mC(m,i)\sum_{j=1}^iS(i,j)·2^{2^{n-m}}·2^{j(n-m)}
$$
时间复杂度$$O(n^3)$$，并不能通过。

我们考虑优化这个过程，观察整个式子，我们把式子变化一下
$$
ans=\sum_{m=0}^n(-1)^mC(n,m)2^{2^{n-m}}\sum_{j=1}^m2^{j(n-m)}\sum_{i=j}^mS(i,j)C(m,i)
$$
考虑化简最后那一个式子，它的组合意义是从$$m$$个中选出若干个分成$$j$$个集合的方案数，我们可以往里面多加一个元素$$0$$，然后把这$$m+1$$个元素分为$$j+1$$组，与$$0$$为一组的我们视为不选（即不出现在集合中），与这个式子等价，那么，最后那个式子就可以变成$$S(m+1,j+1)$$。

整个式子变成了
$$
ans=\sum_{m=0}^n(-1)^mC(n,m)2^{2^{n-m}}\sum_{j=0}^m2^{j(n-m)}S(m +1,j+1)
$$
注意此时$$j$$变为了从$$0$$开始，因为我们会多出来一个集合放不选的。

那么，我们预处理一下$$2^k$$的值和$$S$$直接做就好了。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;
 
const int maxN = 3005;
 
int n, mod;
int S[maxN + 1][maxN + 1], C[maxN + 1][maxN + 1];
int pw[maxN * maxN + 1];
 
inline int ADD(int x, int y) { return x + y >= mod ? x + y - mod : x + y; }
 
inline int SUB(int x, int y) { return x - y < 0 ? x - y + mod : x - y; }
 
inline int mpow(int a, int x, int mod)
{
	int ans = 1;
	while(x)
	{
		if(x & 1) ans = 1ll * ans * a % mod;
		a = 1ll * a * a % mod;
		x >>= 1;
	}
	return ans;
}
 
int main()
{
	scanf("%d %d", &n, &mod);
 
	S[0][0] = C[0][0] = 1;
	for(int i = 1; i <= n + 1; i++)
	{
		C[i][0] = 1;
		for(int j = 1; j <= i; j++)
			C[i][j] = ADD(C[i - 1][j], C[i - 1][j - 1]),
			S[i][j] = ADD(S[i - 1][j - 1], 1ll * j * S[i - 1][j] % mod);
	}
 
	pw[0] = 1;
	for(int i = 1; i <= n * n; i++)
		pw[i] = 2ll * pw[i - 1] % mod;
 
	int ans = 0;
	for(int i = 0; i <= n; i++)
	{
		int res = 0;
		for(int j = 0; j <= i; j++)
			res = ADD(res, 1ll * pw[j * (n - i)] * S[i + 1][j + 1] % mod);
		res = 1ll * res * C[n][i] % mod * mpow(2, mpow(2, n - i, mod - 1), mod) % mod;
		ans = (i & 1) ? SUB(ans, res) : ADD(ans, res);
	}
 
	printf("%d", ans);
	return 0;
}
```

# 总结

从这道题里，我们学到了什么？

首先，如果遇到每个元素都满足什么条件并且不好搞的话，考虑**正难则反**，如果其补集很好算的话就直接算，如果还是不好算的话，考虑**容斥**，钦定一些东西不满足条件，然后别的随便选。

然后，对于一些式子有的时候我们无从下手来化简，这个时候我们可以考虑组合意义，比如我们不选的东西可以看成划分成一个额外的集合，然后钦定那个集合不选，有的时候就能化简复杂度。

另外，一定要搞清楚集合之间是否独立，例如本题中，我们钦定的元素分出来的集合是独立的，两两不同，可以随便选我们未钦定的元素构成的集合，互不干扰，这些关系一定要弄清楚再推式子。

容斥好难啊，还要多加练习！




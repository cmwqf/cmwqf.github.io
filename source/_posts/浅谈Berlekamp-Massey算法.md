---
title: 浅谈Berlekamp-Massey算法
date: 2020-07-18 22:32:35
tags: [线性代数]
categories: [学习笔记]
---

# BM算法

如果我们已知一个数列是线性递推数列（或猜测一个数列是线性递推数列），但是我们并不知道它的线性递推式具体是多少，那么我们可以使用$$Berlekamp-Massey$$算法，简称$$BM$$算法，来计算最短线性递推式。

具体来说，如果已知数列可以写成
$$
a[n]=\sum_{i=1}^mr[i]*a[n - i]\quad(n\ge m+1)
$$
给定长度为$$n$$数列$$a$$，那么我们可以通过$$BM$$计算出这个数列最短的线性递推式（设长度为$$m$$），满足对于$$m+1\le i\le n$$，$$a[i]$$都满足这个规律。

<!--more-->

# 原理

考虑通过不断增加数来修改线性递推数列，设$$R_i$$为第$$i$$版线性递推数列。

假设我们现在已经求出$$a_1,a_2,...a_{i-1}$$的线性递推数列$$r_1,r_2,...r_m$$（记为$$R_{cnt}$$），现在我们要求$$a_1,a_2,...,a_i$$的线性递推数列。

设$$delta_i=\sum_{j=1}^mr[j]*a[n-j]-a[i]$$。

如果$$delta_i=0$$，那么说明$$R_{cnt}$$对$$a_1,a_2,...,a_i$$也成立。

否则，说明我们需要调整$$R_{cnt}$$为$$R_{cnt+1}$$。怎么调整呢？

首先，如果此时$$cnt=0$$，那么说明$$a_i$$是数列中第一个非零数，那么我们只需将$$R_1$$置为$$\{0,0,..,0\}$$（$$i$$个$$0$$）即可。

否则，如果我们能找到另一个线性递推数列，使得这个线性递推数列前$$i-1$$个数都是$$0$$，而第$$i$$个数是$$delta_i$$，那么我们只需将这个线性递推数列和$$R_{cnt}$$加起来就能得到一个合法的$$R_{cnt+1}$$。

其实要构造这样的线性递推数列很简单，我们只需找任意一个历史版本的$$R_k(k<cnt)$$。设$$w=fail_k$$为使得$$R_k$$变得不合法的第一个位置。那么我们设$$mul=\frac{delta_i}{delta_w}$$，并构造$$\{0,0,...,0,mul,-mul*R_k\}$$（前面有$$i-w-1$$个$$0$$）即可，设其长度为$$m$$。

这样，对于$$\forall j<i$$，$$mul*a[j-(i-w)]$$与最后的相抵消，结果为$$0$$。而当$$j=i$$时，结果是$$delta_i$$，因此满足我们所需的数列条件。

因此，$$R_{cnt+1}=\{0,0,...,0,mul,-mul*R_k\}+R_{cnt}$$，长度为$$max(i-w+len_k,len_{cnt})$$。

而我们要求最短，因此只需记录下当前$$len_k-fail_k$$最短的$$R$$即可。

# 注意

用$$BM$$得到的最短线性递推式最好要长度远小于$$\frac{n}{2}$$才结束，否则要再打表。因为如果长度为$$\frac{n}{2}$$，那么相当于$$\frac{n}{2}$$个未知数列$$\frac{n}{2}$$个方程，总能找到解。因此对于任何一个长度为$$n$$随机数列，它的线性递推数列也是$$\frac{n}{2}$$左右，所以只有当长度远小于$$\frac{n}{2}$$时才能大致确定是原数列的线性递推式。如果长度一直在$$\frac{n}{2}$$左右，那么很可能这个数列并不是线性递推数列。

# Code

```c++
inline void BM(int *a, vector<int> &ans)
{
	ans.clear();
	vector<int> lst;
	int w = 0, delta = 0;
	for(int i = 1; i <= n; i++)
	{
		int t = 0;
		for(int j = 0; j < ans.size(); j++)
			t = ADD(t, 1ll * ans[j] * a[i - j - 1] % mod);
		if(t == a[i]) continue;
		if(!w)
		{
			w = i; delta = SUB(a[i], t);
			for(int j = 1; j <= i; j++) ans.push_back(0);
			continue;
		}
		vector<int> now = ans;
		int mul = 1ll * SUB(a[i], t) * mpow(delta, mod - 2) % mod;
		if(i - w + (int)lst.size() > (int)ans.size()) ans.resize(i - w + (int)lst.size(), 0);
		ans[i - w - 1] = ADD(ans[i - w - 1], mul);
		for(int j = 0; j < lst.size(); j++) ans[i - w + j] = SUB(ans[i - w + j], 1ll * mul * lst[j] % mod);
		if((int)now.size() - i < (int)lst.size() - w) delta = SUB(a[i], t), w = i, lst = now;
	}
}
```


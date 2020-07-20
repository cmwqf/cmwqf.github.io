---
title: AGC021F Trinity
date: 2020-04-02 11:28:26
tags: [dp, 组合数学]
categories: [Atcoder]
---

# Description

有一个$$n\times m$$的网格，你要把它每个格子涂成黑或白色，定义$$A_i$$为第$$i$$行第一个涂成黑色的列，$$B_i$$为第$$i$$列第一个涂成黑色的行，$$C_i$$表示第$$i$$列最后一个涂成黑色的行，问有多少种可能的$$(A,B,C)$$组合？

其中$$n\le 8000,m\le 200$$

<!--more-->

# Solution

因为列的限制要多于行的限制，所以我们可以按列的顺序来考虑，每次考虑当前列行的选取情况。

考虑$$dp$$，设$$f[i][j]$$表示前$$i$$列，已经有$$j$$行被选的方案数。

然后我们发现，我们并不好转移当前列的第一个和最后一个选的是什么。

遇到这种情况，我们一般考虑转化一下状态的定义，设$$f[i][j]$$表示前$$i$$列，选了$$j$$行，仅考虑这$$j$$行的相对顺序，并不管具体是哪$$j$$行的方案数，那么我们每次就是要往当前的行中插入一些行。

那么，最后的答案显然是$$ans=\sum_{i=0}^n \binom{n}{i}f[m][i]$$。

然后我们考虑怎样转移。

我们枚举在当前列插入$$k$$个行，计算方案数。

我们假设我们已经插入完了，那么插入完以后，当前列有多少种的选择方案？

首先当前插入的行我们肯定是要选的，那么我们的问题就是原本就存在的行的选择情况。我们发现，只有在**插入的第一个行的上面有多少原来存在的行和插入的最后一行下面有多少原来存在的行才对我们的答案有影响**，不妨设插入的第一个行上面有$$c_1$$个行，插入的最后一个行下面有$$c_2$$个行，那么当前插入方案对答案的贡献就是$$(c_1+1)(c_2+1)$$。

我们将插入行的方案数和对答案的贡献放在一起考虑，我们假设插入的$$k$$个行把原来的$$j$$行分为$$k+1$$段，那么我们有
$$
x_1+x_2+...+x_{k+1}=j\\x_1,x_2,...,x_{k+1}\ge 0
$$
对于每种满足条件的方案数，对答案的贡献为

$$
(x_1+1)(x_{k+1}+1)
$$

这是一个经典的问题了，考虑组合意义，相当于我们要在分出的$$k+1$$个组的第一个和最后一个组再分为$$\ge 0$$的两个部分的方案数。

设
$$
x_1=x_{1,0}+x_{1,1}\\
x_{k+1}=x_{k+1,0}+x_{k+1,1}
$$
那么答案相当于
$$
x_{1,0}+x_{1,1}+x_2+x_3+...+x_{k+1,0}+x_{k+1,1}=j
$$
其中所有元素$$\ge 0$$的整数解个数，那么显然通过组合数学知识，这个方案数等于$$\binom{j+k+2}{k+2}$$。

但是注意，以上的分析仅在$$k>0$$的时候成立，如果$$k=0$$，要单独讨论。

考虑$$k=0$$的情况，首先，如果$$j=0$$，那么贡献为$$1$$；否则，考虑现在$$j$$行，一共有$$j+1$$个空位，我们选择两个空位，这样我们相当于选**较上面那个空位的下面一行，较下面那个空位的上面一行**，这样选的话，能精准覆盖到每一种情况，那么贡献即为$$\binom{j+1}{2}+1$$，$$+1$$表示一个都不选。

那么转移方程就出来了
$$
f[i][j+k]+=\binom{j+k+2}{k+2}*f[i][j]\quad(k>0)\\
f[i][j]+=f[i][j]*(\binom{j+1}{2}+1)
$$
这样时间复杂度为$$O(mn^2)$$显然无法通过本题。

那么我们考虑优化，第二个式子复杂度正确，不需要优化，那么我们看第一个式子。

第一个式子相当于
$$
f[i][j]=\sum_{k=0}^{j-1}\binom{j+2}{j+2-k}*f[i][k]\\
=(j+2)!\sum_{k=0}^{j-1}\frac{1}{(j+2-k)!}*\frac{f[i][k]}{k!}
$$
这显然是一个卷积形式，相当于
$$
c[n]=\sum_{i=0}^{n-1}\frac{1}{(n-1-(i-3))!}*\frac{a[i]}{i!}
$$
直接$$NTT$$优化即可，时间复杂度$$O(mnlog_2n)$$。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxN = 4e4 + 10, mod = 998244353, g = 3, gi = 332748118;

int n, m;
int lim, cnt, r[maxN + 1];
int f[205][8005];
int A[maxN + 1], B[maxN + 1];
int fac[maxN + 1], inv[maxN + 1];

inline int ADD(int x, int y) { return x + y >= mod ? x + y - mod : x + y; }

inline int SUB(int x, int y) { return x - y < 0 ? x - y + mod : x - y; }

inline int mpow(int a, int x)
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

inline void NTT(int *a, int type)
{
	for(int i = 0; i < lim; i++)
		if(i < r[i]) swap(a[i], a[ r[i] ]);
	for(int mid = 1; mid < lim; mid <<= 1)
	{
		int wn = mpow(type == 1 ? g : gi, (mod - 1) / (mid << 1));
		for(int i = 0; i < lim; i += (mid << 1))
		{
			int w = 1;
			for(int j = 0; j < mid; j++, w = 1ll * w * wn % mod)
			{
				int x = a[i + j], y = 1ll * a[i + mid + j] * w % mod;
				a[i + j] = ADD(x, y); a[i + mid + j] = SUB(x, y);
			}
		}
	}
	if(type == -1)
	{
		int inv = mpow(lim, mod - 2);
		for(int i = 0; i < lim; i++) a[i] = 1ll * a[i] * inv % mod;
	}
}

inline int C(int n, int m)
{
	if(n < m) return 0;
	return 1ll * fac[n] * inv[m] % mod * inv[n - m] % mod;
}

int main()
{
	scanf("%d %d", &n, &m);

	fac[0] = 1;
	for(int i = 1; i <= n + 5; i++) fac[i] = 1ll * fac[i - 1] * i % mod;
	inv[n + 5] = mpow(fac[n + 5], mod - 2);
	for(int i = n + 4; i >= 0; i--) inv[i] = 1ll * inv[i + 1] * (i + 1) % mod;

	lim = 1, cnt = 0;
	while(lim <= n * 2) lim <<= 1, cnt ++;
	for(int i = 0; i < lim; i++) r[i] = (r[i >> 1] >> 1) | ((i & 1) << cnt - 1);

	for(int i = 0; i <= n; i++) B[i] = inv[i + 3];
	NTT(B, 1);

	f[0][0] = 1;
	for(int i = 1; i <= m; i++)
	{
		for(int j = 0; j <= n; j++) A[j] = 1ll * f[i - 1][j] * inv[j] % mod;
		
		NTT(A, 1);
		for(int j = 0; j < lim; j++) A[j] = 1ll * A[j] * B[j] % mod;
		NTT(A, -1);

		f[i][0] = f[i - 1][0];
		for(int j = 1; j <= n; j++) 
			f[i][j] = ADD(1ll * fac[j + 2] * A[j - 1] % mod, 1ll * ADD(C(j + 1, 2), 1) * f[i - 1][j] % mod);

		for(int j = 0; j < lim; j++) A[j] = 0;
	}

	int ans = 0;
	for(int i = 0; i <= n; i++) ans = ADD(ans, 1ll * f[m][i] * C(n, i) % mod);

	printf("%d", ans);
	return 0;
}

```

# 总结

我们对于两个维度限制问题，一般按照限制多的那一维度的顺序进行$$dp$$，每次考虑当前维度的选择方案。

对于网格问题，也可以使用**考虑相对顺序，每次插入的方法**。

实际上，如果我们每次的决策依赖于我们已经选择的东西的顺序情况，但并不知道具体选择了什么东西的时候，就可以考虑插入的方法。

我们按照一定顺序进行插入，就是为了去掉一些限制，比如我们按照从大到小插入，我们就钦定了当前插入的元素是所有的元素中最小的，就去掉了大小限制，再比如这道题，我们每次插入，就能知道已经选择的行和当前插入行的顺序关系，便于做题。

这个思想非常重要，考试的时候如果没有思路也许可以往插入的方向想一想，有时能直接去掉一些限制。
---
title: '[LOJ3284]Exercise'
date: 2020-04-10 12:02:34
tags: [组合数学, dp, 数论, 容斥原理]
categories: [LOJ]
---

# Description

给定$$n,m$$，设一个$$1$$到$$n$$的置换$$P$$，定义$$f(P)$$是满足$$P^x=I$$的最小的$$x$$（$$I$$是单位置换），求$$\prod_P f(P)\mod m$$。

其中$$n\le 7500,1e8\le m\le 1e9+7$$，且$$m$$为质数。

**本题来源于USACO 2020 US Open Platinum T2。**

<!--more-->

# Solution

注意到$$f(P)$$实际上就是$$P$$所有循环长度的最小公倍数。

直接求最小公倍数似乎无从下手，但题目要求的是乘积，所以我们可以枚举质数，然后计算每一个质数对答案的贡献。

对于一个$$p$$，定义$$ord_p(x)$$为$$x$$质因子分解后$$p$$的指数，那么我们要计算的是
$$
\sum_Smax_{a\in S} ord_p(a)
$$
对于$$S$$，我们要满足
$$
\sum_{a\in S}a=n
$$
根据套路，我们把换成
$$
\sum_{k\ge 1}k*\sum_S[max_{a\in S}ord_p(a)=k]
$$
这个时候，我们肯定是要对这个进行$$dp$$的，我们每次要枚举加入的数。

我们发现这个$$max$$对复杂度非常不友好，要$$O(n)$$地进行枚举，而如果改成$$min$$的话，我们枚举量就可以缩减为$$\frac{n}{p^k}$$，大大减少了复杂度。怎么把$$max$$改成$$min$$呢？$$min-max$$容斥！

想不到吧，$$min-max$$容斥不仅能用在期望上，有时还能优化复杂度！

那么我们就变为
$$
\sum_{k\ge 1}k*\sum_S\sum_{T\subset S}(-1)^{|T|-1}[min_{a\in T}ord_p(a)=k]\\
=\sum_{k\ge 1}k*\sum_T(-1)^{|T|-1}[min_{a\in T}ord_p(a)=k]*coef(T)
$$
这个$$coef(T)$$就是$$T$$对答案的贡献系数，我们后面再说。

我们发现这个$$k$$非常地不好弄，那么我们还有一个常用办法，把式子转变成：
$$
\sum_{k\ge 1}\sum_T(-1)^{|T|-1}[ord_p(a)\ge k,\forall a\in T]*coef(T)
$$
这个时候，一个$$T$$满足$$min(ord_p(a))=k$$，它对答案依然会产生$$k$$的贡献。这是非常常用的方法。这也是我们为什么把$$max$$转为$$min$$的重要原因！

我们现在来看$$coef(T)$$，实际上，它的含义为剩下的随便变成若干环排列。我们记$$sum(T)$$为$$T$$中所有元素的和，那么首先我们要从$$n$$个中选出$$sum(T)$$个，先乘以$$\binom{n}{sum(T)}$$，现在来看剩下$$n-sum(T)$$个数构成若干环排列的方案数。

我们设$$g(m)$$表示$$m$$个数分成若干环排列的方案数，那么我们每次考虑加入一个数，和第一类斯特林数推导类似的思想，我们考虑它放在哪：一种情况，它自立门户，单独成为一个环排列，另外一种情况，它放在前面某个数的一侧。那么递推式就有了：
$$
g(m)=(m-1)*g(m-1)+g(m-1)*1=m*g(m-1)
$$
根据$$g(1)=1$$，我们发现$$g(m)=m!$$，这也告诉我们第一类斯特林数第$$i$$行数的和为$$i!$$。

那么我们就有
$$
coef(T)=\binom{n}{sum(T)}*(n-sum(T))!
$$
我们发现这个$$coef(T)$$只与$$sum(T)$$有关，那么我们就可以枚举$$k$$进行$$dp$$了。

下面设$$s=p^k$$，我们相当于钦定每个环排列大小都是$$s$$的倍数来计算方案。

我们设$$f[i]$$表示$$sum(T)=is$$时的容斥系数。现在我们考虑加入一个环排列，首先容斥系数显然取反。然后为了避免算重，根据计数技巧，我们枚举与$$1$$在一起的那个环排列，一个大小为$$m$$的集合的环排列个数显然是$$(m-1)!$$，那么转移方程就出来了。
$$
f[i]=-\sum_{j=1}^i \binom{i*s-1}{j*s-1}*(j*s-1)!*f[i-j]
$$
考虑时间复杂度，我们相当于是对于所有的质数$$p$$对于时间的贡献为
$$
\sum_{p^k\le n}(\lfloor\frac{n}{p^k}\rfloor)^2\le n^2*\sum_k \frac{1}{p^{2k}}=O(\frac{n^2}{p^2})
$$
那么总复杂度为
$$
n^2*\sum_p\frac{1}{p^2}\le n^2
$$
所以复杂度为$$O(n^2)$$。

另外注意，我们在$$dp$$指数，所以$$dp$$值，组合数，阶乘的模数都是$$\phi(mod)=mod-1$$，而我们做快速幂的时候模数是$$mod$$，不要混淆。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxN = 7505;

int n, mod, ans;
int f[maxN + 1], C[maxN + 1][maxN + 1];
int fac[maxN + 1];
bool flag[maxN + 1];

inline int ADD(int x, int y) { return x + y >= mod ? x + y - mod : x + y; }

inline int SUB(int x, int y) { return x - y < 0 ? x - y + mod : x - y; }

inline int mpow(int a, int x)
{
	int ans = 1;
	while(x)
	{
		if(x & 1) ans = 1ll * ans * a % (mod + 1);
		a = 1ll * a * a % (mod + 1);
		x >>= 1;
	}
	return ans;
}

inline int calc(int k)
{
	int m = n / k, res = 0;
	f[0] = mod - 1;
	for(int i = 1; i <= m; i++)
	{
		f[i] = 0;
		for(int j = 1; j <= i; j++)
			f[i] = SUB(f[i], 1ll * C[i * k - 1][j * k - 1] * fac[j * k - 1] % mod * f[i - j] % mod); 
	}
	for(int i = 1; i <= m; i++)
		res = ADD(res, 1ll * C[n][i * k] * fac[n - i * k] % mod * f[i] % mod);
	return res;
}

int main()
{
	scanf("%d %d", &n, &mod);
	mod --;

	fac[0] = 1;
	for(int i = 1; i <= n; i++) fac[i] = 1ll * fac[i - 1] * i % mod;

	for(int i = 0; i <= n; i++)
	{
		C[i][0] = 1;
		for(int j = 1; j <= i; j++) C[i][j] = ADD(C[i - 1][j], C[i - 1][j - 1]);
	}

	ans = 1;
	for(int i = 2; i <= n; i++)
		if(!flag[i])
		{
			for(int j = i + i; j <= n; j += i) flag[j] = true;
			for(int j = i; j <= n; j *= i)
				ans = 1ll * ans * mpow(i, calc(j)) % (mod + 1);
		}

	printf("%d\n", ans);
	return 0;
}
```

# 总结

首先，对于一些特定的题目，会有$$min$$比$$max$$更适合解题或$$max$$比$$min$$更适合解题的情况，这个时候也可以考虑$$min-max$$容斥。

其次，对于一些情况（比如本题），我们可以用$$=$$转$$\ge $$的容斥，既放宽松了条件，又去掉了一个系数，十分巧妙。

另外，我们在$$dp$$有关集合的时候，为了防止算重，我们会枚举与某个特定的数在一起的那个元素（比如本题枚举与$$1$$在一起的环排列），这个技巧也要牢记。
---
title: '[BZOJ3027]Sweet'
date: 2019-07-25 08:34:11
tags: [生成函数,组合数学]
categories: [BZOJ]
---

# Description

有$$n$$种糖果，每种有$$m_i$$个，求一共吃$$a->b$$个糖果的方案数之和（对2004取模）。

<!--more-->

# Solution

考虑生成函数，对于每种糖果，它的生成函数是
$$
1+x+x^2+...+x^{m_i}=\frac{1-x^{m_i+1}}{1-x}
$$
所有的乘起来，再变换一下，就是
$$
(1+x+x^2+...)^n\prod_{i=1}^n(1-x^{m_i+1})
$$
我们要求的，就是这个式子$$x$$次数为$$a->b$$的前面的系数之和。

怎么办呢？我们发现，后面那个式子最多只有$$2^n$$项，所以我们可以暴力展开，$$dfs$$枚举每一项。

注意前面那个式子$$x^i$$前面的系数是$$C_{i+n-1}^{n-1}$$

假设我们现在枚举到了$$t*x^k$$，那么这一项对答案的贡献就是：
$$
t*(C_{a-k+n-1}^{n-1}+...+C_{b-k+n-1}^{n-1})
$$
我们知道
$$
C_{n-1}^{n-1}+...+C_{i+n-1}^{n-1}=C_{i+n}^n
$$
所以对答案的贡献可以写成
$$
t*(C_{b-k+n}^n-C_{a-k+n-1}^n)
$$
那么，问题来了，因为2004是个合数，所以可能不存在逆元，那么我们怎么算后面这个东西呢？

我们考虑暴力计算$$(n-m+1)*...*n$$，这样，我们就剩下要除以一个$$n!$$了。

首先，前面那个式子一定有$$n!$$这个因子，设为$$(k*mod+r)*n!$$，那么答案应该就是$$r$$，现在因为我们不能直接除以$$n!$$，所以我们改为将它模$$mod*n!$$。

这样，我们算出来的答案就是$$r*n!$$，然后我们再除以$$n!$$就好了。

# Code

```c++
#include <cstdio>
using namespace std;

#define int long long

const int maxN = 500 + 10;

int m[maxN + 1], mod = 2004;
int tmp = 1, ans;
int n, a, b;

inline int C(int n, int m)
{
	if(n < m) return 0;
	int ans = 1;
	for(int i = n - m + 1; i <= n; i++) ans = ans * i % mod;
	return ans;
}

inline void dfs(int x, int lim, int num)
{
	if(lim > b) return;
	if(x > n)
	{
		ans += num * ( C(b - lim + n, n) - C(a - lim + n - 1, n) ) % mod;
		ans %= mod;
		return;
	}
	dfs(x + 1, lim, num);
	dfs(x + 1, lim + m[x] + 1, -num);
} 

#undef int
int main()
#define int long long
{
	scanf("%lld %lld %lld", &n, &a, &b);
	for(int i = 1; i <= n; i++) scanf("%lld", &m[i]);
	for(int i = 1; i <= n; i++) tmp *= i;
	mod *= tmp;
	dfs(1, 0, 1);
	ans /= tmp;
	printf("%lld", (ans % 2004 + 2004) % 2004);
	return 0;
}
```


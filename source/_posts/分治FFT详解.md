---
title: 分治FFT详解
date: 2019-02-18 16:40:57
tags: [多项式]
categories: [学习笔记]
---

# 什么是分治FFT

分治FFT用于快速计算这样一类式子：
$$
f[n]=\sum_{i=0}^{n-1}f[i]g[n-i]
$$
与普通的FFT不同，这个式子是用自己来更新自己，所以如果暴力用FFT，时间复杂度为$$O(n^2logn)$$，因此，我们考虑用分治FFT来解决这类问题。

<!--more-->

# 分治FFT的实现

分治，指的就是cdq分治（运用的是这种思想）：

就是我们考虑先计算出$$f[l],f[l+1],...,f[mid]$$的值，再计算它对$$f[mid+1],...,f[r]$$的贡献，最终完成计算。

由于计算l~mid的值是递归的事情，所以我们只要考虑$$f[l...mid]$$对于$$f[mid+1...r]$$的贡献即可。

首先，我们考虑，在$$[mid+1,r]$$的每一个数$$x$$,$$[l,mid]$$区间对其贡献为：
$$
w[x]=\sum_{i=l}^{mid}f[i]g[x-i]
$$
我们将求和的范围扩大到$$[l,x-1]$$，因为$$f[mid+1]->f[x-1]$$都是0，所以对式子的值并无影响。
$$
w[x]=\sum_{i=l}^{x-1}f[i]g[x-i]
$$
这时，我们考虑计算右边那个式子，我们可以设$$a[i]=f[i+l],b[i]=g[i+1]$$

这个式子就变化为
$$
\sum_{i=0}^{x-l-1}a[i]b[x-l-1-i]
$$
由卷积的知识可以得知，这个式子可以看成
$$
c[x-l-1]=\sum_{i=0}^{x-l-1}a[i]b[x-l-1-i]
$$
这个式子，就可以用FFT来完成了。结合上面那个$$w[x]$$的式子，可以得到：
$$
w[x]=c[x-l-1]=\sum_{i=0}^{x-l-1}a[i]b[x-l-1-i]
$$
所以，我们在做完卷积之后，直接：
$$
w[x]=a[x-l-1](a为卷积后的数组)
$$
最后，将贡献加上去就可以了：
$$
f[x]+=w[x]，即f[x]+=a[x-l-1]
$$
因此，我们所需要计算的数组的长度为$$r-l-1$$，时间复杂度为$$O(nlog^2n)$$

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxN = 1 << 20 | 5, mod = 998244353;

int n;
int a[maxN + 1], b[maxN + 1];
int A[maxN + 1], B[maxN + 1];
int wn[maxN + 1], R[maxN + 1];

inline int read()
{
	int num = 0, f = 1;
	char ch = getchar();
	while( !isdigit( ch ) ) { if(ch == '-') f = -1; ch = getchar(); }
	while( isdigit( ch ) ) num = (num << 3) + (num << 1) + (ch ^ 48), ch = getchar();
	return num * f;
}

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

inline void pre()
{
	int lim = 1, cnt = 0;
	while(lim <= 2 * n) lim <<= 1, cnt ++;
	for(int mid = 1; mid < lim; mid <<= 1)
	{
		int W = mpow(3, (mod - 1) / (mid << 1));
		wn[mid] = 1;
		for(int j = 1; j < mid; j++) wn[mid + j] = 1ll * wn[mid + j - 1] * W % mod;
	}
}

inline void NTT(int *a, int type, int lim)
{
	for(int i = 0; i < lim; i++)
		if(i < R[i]) swap(a[i], a[ R[i] ]);
	for(int mid = 1; mid < lim; mid <<= 1)
		for(int i = 0; i < lim; i += (mid << 1))
			for(int j = 0; j < mid; j++)
			{
				int x = a[i + j], y = 1ll * a[i + mid + j] * wn[mid + j] % mod;
				a[i + j] = ADD(x, y); a[i + mid + j] = SUB(x, y);
			}
	if(type == -1)
	{
		int INV = mpow(lim, mod - 2);
		for(int i = 0; i < lim; i++) a[i] = 1ll * a[i] * INV % mod;
		reverse(a + 1, a + lim);
	}
}

inline void solve(int l, int r)
{
	if(l == r) return;

	int mid = (l + r) >> 1;
	solve(l, mid);

	for(int i = l; i <= mid; i++) A[i - l] = a[i];
	for(int i = 1; i <= r - l; i++) B[i - 1] = b[i];

	int lim = 1, cnt = 0;
	while(lim <= r - l + mid - l + 1) lim <<= 1, cnt ++;
	for(int i = 0; i < lim; i++) R[i] = (R[i >> 1] >> 1) | ((i & 1) << cnt - 1);
	
	NTT(A, 1, lim); NTT(B, 1, lim);
	for(int i = 0; i < lim; i++) A[i] = 1ll * A[i] * B[i] % mod;
	NTT(A, -1, lim);

	for(int i = mid - l; i <= r - l - 1; i++) a[i + l + 1] = ADD(a[i + l + 1], A[i]);	

	for(int i = 0; i < lim; i++) A[i] = B[i] = 0;

	solve(mid + 1, r);
}

int main()
{
	n = read();
	for(int i = 1; i <= n - 1; i++) b[i] = read();

	pre();
	
	a[0] = 1;
	solve(0, n - 1);

	for(int i = 0; i < n; i++) printf("%d ", a[i]);
	puts("");
	return 0;
}
```






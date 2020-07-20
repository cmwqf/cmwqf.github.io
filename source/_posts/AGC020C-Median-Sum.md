---
title: AGC020C Median Sum
date: 2020-01-08 16:54:58
tags: [dp, bitset]
categories: [Atcoder]
---

# Description

给定$$n$$个数$$a_i$$，求这$$n$$个数构成的$$2^n-1$$个子集的和的中位数。

其中$$n\le 2000,a_i\le 2000$$

<!--more-->

# Solution

$$Atcoder$$的题面永远那么简单，思维难度依然那么高。。。

看到这个中位数就感到很头疼，这么大的范围，感觉可能就是一个结论题。

确实是结论题，只是想不到。。。

设$$sum$$表示这些数的和。

考虑任何一个$$\sum q_i< \frac{1}{2}sum$$的非空也非全集的子集$${q}$$，那么它的补集，记为$$q$$，因为$$\sum p_i + \sum q_i = sum$$，必然有$$\sum q_i > \frac{1}{2}sum$$，所以，如果把所有和不等于$$\frac{1}{2}sum$$的子集分为$$<\frac{1}{2}sum$$及$$>\frac{1}{2}sum$$两组，**两组之间的元素是一一对应的**。也就是说，**两组中的元素个数是相等的！**

**一一对应推到出个数相等，这是数学中的常用技巧，也适用于信息竞赛中。**

然后，等于$$\frac{1}{2}sum$$的子集个数显然也是偶数个，因此，这个集合的中位数就相当于在$$\frac{1}{2}sum$$的中间位置划了一条线，前面和后面的元素个数相等。

最后，因为最终的集合个数又加了一个总的和（加在最后），所以中位数就是所有出现的和中第一个$$\ge \frac{1}{2}sum$$的数。注意，因为$$sum$$可能是奇数，所以应该是$$\ge (sum+1)/2$$。

那么，考虑怎么维护最终所有可能出现的和的集合？

动态规划？超时。而$$BUG$$般的存在$$bitset$$就能完美地在$$O(\frac{n^2\times max(a_i) }{w})$$的时间解决这道题了。

哎，思维还是不太行呢。。。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

const int N = 2010;

int n, sum, x;

bitset<N * N> s;

int main()
{
	scanf("%d", &n);
	s[0] = 1;
	for(int i = 1; i <= n; i++)
		scanf("%d", &x), s = s | (s << x), sum += x;
	sum = (sum + 1) / 2;
	for(int i = 1; i < N * N; i++)
		if(s[i] && i >= sum) { printf("%d", i); return 0; }
	return 0;
}
```


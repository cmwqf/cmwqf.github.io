---
title: AGC011F Train Service Planning
date: 2020-04-03 10:19:49
tags: [贪心, dp]
categories: [Atcoder]
---

# Description

有一个$$n+1$$个车站的铁路，分别为$$0,1,...,n$$号，显然铁路被分为$$n$$段，设$$a_i$$为火车从第$$i-1$$个车站到第$$i$$个车站所需时间（也是第$$i$$个车站到第$$i-1$$个车站的所需时间），每段铁路还有一个参数$$b_i=1/2$$表示这段路是单向道还是双向道，如果是单向道，同一时间不能被两辆不同方向的火车经过，如果是双向道就没有限制。

现在有两种车，一种从$$0$$开到$$n$$，另外一种从$$n$$开到$$0$$，且每隔$$k$$单位时间就会发一辆车（两种都发），我们的任务是调整两种车在每个站台的停留时间，使得不存在某个时刻两种车在同一个单向道上相遇，求满足这个条件的情况下，$$0->n$$的时间与$$n->0$$的时间之和的最小值。若无法完成，则输出$$-1$$。

其中$$n\le 1e5,k,a_i\le 1e9,b_i\le 2$$。

<!--more-->

# Solution

非常巧妙的一道题。

首先，我们注意到因为每隔$$k$$发一次车，所以整个问题我们可以看成在$$mod\quad k$$意义下进行的。

然后，我们发现整个问题可以转化为区间问题，即有一些长度固定的区间，然后我们的限制是在模意义下，一些区间不能相交。

具体地，我们设$$p_i$$为第一种车在第$$i$$个车站的停留时间（$$0\le i \le n$$），设$$q_i$$为第二种车在第$$i$$个车站的停留时间，然后设$$P_i$$表示$$p_0,...p_i$$的和，$$Q_i$$同理，$$A_i$$即$$a$$的前缀和。

我们发现第一种车的区间是$$p$$和$$a$$的前缀和的形式，第二种车的每个区间却是$$q$$和$$a$$的后缀和的形式，非常不好处理。

但是我们发现一个好处是整个问题是在取模的意义下进行的。

在时间轴上取模，就相当于拿一个长度为$$k$$的窗口在时间轴上滑动，不管窗口在什么位置，截到的时间及其上面发生的事件都是等价的。

我们设窗口截到的最右边是整个事件结束，那么$$q_0$$的意义相当于第二种车到达$$0$$号站后再等待$$q_0$$时间才得以结束。

为了与第一种车的区间形式相似，我们考虑对第二种车进行时光倒流，把第二种车也看成从$$0->n$$，然后看成它在$$0$$号车站等待了$$-q_0$$单位时间，然后到达$$1$$号车站的时间为$$-q_0-a_1$$，以此类推。

通过这样的操作，我们将第一种车和第二种车的区间形式都统一为前缀和的形式。

更具体地，第一种车在第$$i-1$$到第$$i$$个车站的时间区间和第二种车从第$$i$$个车站到第$$i - 1$$个的时间区间分别为为
$$
(P_{i-1}+A_{i-1},P_{i-1}+A_i)\\
(-Q_{i-1}-A_i,-Q_{i-1}-A_{i-1})
$$
这样看起来形式非常统一，我们后面也会很好处理。

现在，我们就是要求在取模意义下，这两个区间不相交。因为区间相交比较好算（好理解），我们先转化为区间相交，再取补集。

区间相交，即其中一个区间的端点在另一个区间内。

我们列出以下不等式
$$
-Q_{i-1}-A_i<P_{i-1}+A_{i-1}<-Q_{i-1}-A_{i-1}\\
-Q_{i-1}-A_i<P_{i-1}+A_i<-Q_{i-1}-A_{i-1}
$$
两个不等式是或的关系，即只要一个成立即可。

这两个不等式肯定是在模的意义下成立，但是在模的意义下理解不等式又比较困难，我们这么来思考模意义下不等式的含义。

实际上，在模意义下，$$a<b<c$$的意义为在$$(a,c)$$中存在一个数，与$$b$$同余。这样理解起来就要好多了！

那么显然$$a<b<c$$等价于$$a+d<b+d<c+d$$（考虑上面所说的意义即可），即这个不等式可以按照解普通不等式的方法来解。

那么解得
$$
-2A_i<P_{i-1}+Q_{i-1}<-2A_{i-1}
$$
那么其补集就是在模意义下，
$$
P_{i-1}+Q_{i-1}\in [-2A_{i-1},-2A_i]
$$
这样一来，因为$$A$$是已知的，所以我们就能确定$$P_i+Q_i$$的范围。

因为我们最终要求最小化的是
$$
P_{n-1}+Q_{n-1}-p_0-q_0
$$
所以问题就转化成十分可做的形式了！

这样，问题转化为：

有一个数列$$t$$，满足$$t[i]\ge t[i-1]$$，然后对于某些特定的$$i$$（$$b_i=1$$），要满足$$t[i]\quad mod\quad k\in [L[i],R[i]]$$。求$$t_n-t_1$$的最小值。

我们考虑用$$dp$$来解决，设$$f[i][j]$$表示处理了前$$i$$个，当前$$t[i]\quad mod\quad k=j$$时最小的$$t[i]$$。（对于$$-t_1$$的限制可以直接把$$f[1]$$中满足条件的变成$$0$$，别的变成$$inf$$），那么转移的时候，首先如果本来就在$$[L[i],R[i]]$$中的自然不会改变，其他的因为只能往一个方向走，必然是走到$$L[i]$$处，那么用不在区间中的步数最小值来更新$$L[i]$$的值，并把不在区间中的全部赋值为$$inf$$即可。

这样复杂度显然飞了，我们发现几个显然的性质：

因为每次必然会走到$$L[i]$$处或者不动，所以$$j$$的取值只有$$2n$$种。

然后我们发现这个东西显然可以用一棵线段树方便地维护，所以直接线段树优化一下即可，记得线段树种记得是当前最小的步数减去当前位置的值，用来更新后面的答案。

最后输出线段树中最小值加上$$2A[n]$$即可。

时间复杂度$$O(nlog_2n)$$，记得特判如果$$b[i]==1$$且$$2a[i]>k$$，输出$$-1$$，这个根据前面区间的分析很容易得到。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

#define LL long long

const int maxN = 2e5 + 10;
const LL inf = 1e18;

struct Segment
{
	LL mn;
	bool lazy;
}tree[maxN * 4 + 1];

int n, K, cnt;
int a[maxN + 1], b[maxN + 1];
int L[maxN + 1], R[maxN + 1], tmp[maxN + 1];
LL sum[maxN + 1], ans;

inline int read()
{
	int num = 0, f = 1;
	char ch = getchar();
	while( !isdigit( ch ) ) { if(ch == '-') f = -1; ch = getchar(); }
	while( isdigit( ch ) ) num = (num << 3) + (num << 1) + (ch ^ 48), ch = getchar();
	return num * f;
}

inline void pushup(int node)
{
	tree[node].mn = min(tree[node << 1].mn, tree[node << 1 | 1].mn);
}

inline void pushdown(int node)
{
	tree[node << 1].mn = tree[node << 1 | 1].mn = inf;
	tree[node << 1].lazy = tree[node << 1 | 1].lazy = true;
	tree[node].lazy = false;
}

inline void build(int node, int l, int r)
{
	tree[node].lazy = false;
	if(l == r) { tree[node].mn = -tmp[l]; return; }
	int mid = (l + r) >> 1;
	build(node << 1, l, mid);
	build(node << 1 | 1, mid + 1, r);
	pushup(node);
}

inline void modify(int node, int l, int r, int x, int y)
{
	if(x > y) return;
	if(x <= l && r <= y)
	{
		tree[node].mn = inf;
		tree[node].lazy = true;
		return;
	}
	if(tree[node].lazy) pushdown(node);
	int mid = (l + r) >> 1;
	if(x <= mid) modify(node << 1, l, mid, x, y);
	if(y > mid) modify(node << 1 | 1, mid + 1, r, x, y);
	pushup(node);
}

inline void change(int node, int l, int r, int x, LL num)
{
	if(l == r) { tree[node].mn = min(tree[node].mn, num); return; }
	if(tree[node].lazy) pushdown(node);
	int mid = (l + r) >> 1;
	if(x <= mid) change(node << 1, l, mid, x, num);
	else change(node << 1 | 1, mid + 1, r, x, num);
	pushup(node);
}

inline LL query(int node, int l, int r, int x, int y)
{
	if(x > y) return inf;
	if(x <= l && r <= y) return tree[node].mn;
	if(tree[node].lazy) pushdown(node);
	int mid = (l + r) >> 1; LL ans = inf;
	if(x <= mid) ans = min(ans, query(node << 1, l, mid, x, y));
	if(y > mid) ans = min(ans, query(node << 1 | 1, mid + 1, r, x, y));
	return ans;
}

inline void get(int node, int l, int r)
{
	if(l == r) { ans = min(ans, tree[node].mn + tmp[l]); return; }
	if(tree[node].lazy) pushdown(node);
	int mid = (l + r) >> 1;
	get(node << 1, l, mid);
	get(node << 1 | 1, mid + 1, r);
}

int main()
{
	n = read(), K = read();
	for(int i = 1; i <= n; i++)
	{
		a[i] = read(), b[i] = read();
		sum[i] = sum[i - 1] + a[i];
		if(b[i] == 1) L[i] = (K - 2ll * sum[i - 1] % K) % K, R[i] = (K - 2ll * sum[i] % K) % K;
		else L[i] = 0, R[i] = K - 1;
		if(b[i] == 1 && 2ll * a[i] > K)
		{
			puts("-1");
			return 0;
		}
		tmp[++ cnt] = L[i]; tmp[++ cnt] = R[i];
	}

	sort(tmp + 1, tmp + cnt + 1);
	cnt = unique(tmp + 1, tmp + cnt + 1) - tmp - 1;

	build(1, 1, cnt);

	for(int i = 1; i <= n; i++)
	{
		if(b[i] == 2) continue;
		int l = lower_bound(tmp + 1, tmp + cnt + 1, L[i]) - tmp, r = lower_bound(tmp + 1, tmp + cnt + 1, R[i]) - tmp;
		if(l <= r)
		{
			LL t = min(query(1, 1, cnt, 1, l - 1) + tmp[l], query(1, 1, cnt, r + 1, cnt) + K + tmp[l]);
			modify(1, 1, cnt, 1, l - 1); modify(1, 1, cnt, r + 1, cnt);
			change(1, 1, cnt, l, t - tmp[l]);
		}
		else
		{
			LL t = query(1, 1, cnt, r + 1, l - 1) + tmp[l];
			modify(1, 1, cnt, r + 1, l - 1);
			change(1, 1, cnt, l, t - tmp[l]);
		}
	}

	ans = inf;
	get(1, 1, cnt);

	printf("%lld", ans + 2ll * sum[n]);
	return 0;
}
```

# 总结

$$Atcoder$$这种题，一般都要问题转化。

遇到一些列式子的情况，我们要想办法把式子转化为类似的形式，有利于后面化简（例如本题考虑时光倒流），这是本题最巧妙的思想！

然后对于取模意义下的区间一般都转化成判断相交并转化为不等式问题，一定要多加理解取模意义下的不等式的含义。
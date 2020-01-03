---
title: AGC009E Eternal Average
date: 2020-01-03 16:02:41
tags: [dp]
categories: [Atcoder]
---

# Descrption

有$n$个0和$m$个1，每次选出$k$个数删去，并把它们的平均数加入，最后只剩下一个数字（保证$k-1|n+m-1$），问最后剩下的数字有多少种可能？

<!--more-->

# Solution

又是神仙题一道，$Atcoder$上的$dp$怎么都那么神仙啊。。。

考虑构造一个树形结构，有$n+m$个叶子节点，每次操作相当于给某$k$个点连上一个祖先，权值为其孩子的权值的平均数，显然，最后的答案是根节点的值。

考虑每个叶子节点的贡献，假设$n$个$0$叶子标号为$x1,x2,...xn$，$m$个$1$叶子标号为$y1,y2,...ym$，那么根节点的值就是$\sum k^{-yi}$。这个想一想就知道了。

其实想不到树形结构也没有关系，依然是考虑每个数$i$的贡献，若它被选了$t$次，那么它的贡献就是$\frac{a[i]}{k^t}$。其中$a[i]$是这个数的值。

然而后面的就不好想了，我们怎么计算这个东西的值呢？

看到这样一个固定的值$k$的若干次方和，就应该想到转化为$k$进制数来统计。我们只要统计满足条件的$k$进制小数个数即可。

问题在于，满足什么条件的$k$进制小数才是合法的呢？

我们假设这个$k$进制小数有$len$位，为$0.c_1c_2c_3...c_{len}$，那么首先，很显然地$\sum c_i\le m$（虽然显然，但是还是难以想到这是一个限制条件，可能这就是这一类题的套路吧），还有，如果不考虑进位，可以得知$\sum c_i=m$，然后，考虑每次进位，我们会损失掉$k$个数，但是我们会在前面一位加上1个数，也就是说，每次进位，我们会损失$k-1$个数，那么，另外一个限制条件是$\sum c_i\equiv m\quad (mod\quad k-1)$。

于是，我们就可以用$dp$，设$dp[i][j][0/1]$表示前$i$位，和为$j$，最后一位为$0$或其他的方案数即可。为什么要有最后一位？因为我们统计的时候最后一位一定是非零的，所以要单独拿出来。然后$dp$方程很显然，前缀和优化一下即可。

# 总结

那么，这一道题，我们学到了什么？

首先，对于一些元素进行运算的结果之类的问题，我们可以考虑每个元素的贡献，就会简单很多。

对于形如$\sum_{i=1}^n k^{t_i}$的不同结果数的形式，我们往往把它对应到一个$k$进制数上去统计，而统计的条件是$\sum c_i \le n$，$\sum c_i \equiv n\quad (mod\quad k-1)$，这一点如果没做过这类题，还是挺难想的。

总之，$Atcoder$上的$dp$题思维难度都很高，遇到没有思路的题目，想一想转化模型，比如建图，构树之类的，也许就会豁然开朗。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;
 
const int maxN = 2005, mod = 1e9 + 7;
 
int n, m, k, len, ans;
int f[maxN * 2 + 1][maxN + 1][2], sum[maxN * 2 + 1][maxN + 1];
 
inline int ADD(int x, int y) { return x + y >= mod ? x + y - mod : x + y; }
 
inline int SUB(int x, int y) { return x - y < 0 ? x - y + mod : x - y; }
 
int main()
{
	scanf("%d %d %d", &n, &m, &k);
	len = (n + m - 1) / (k - 1);
	f[0][0][0] = 1;
	for(int i = 0; i <= m; i++) sum[0][i] = 1;
	for(int i = 1; i <= len; i++)
		for(int j = 0; j <= m; j++)
		{
			f[i][j][0] = ADD(f[i - 1][j][0], f[i - 1][j][1]);
			if(j) f[i][j][1] = SUB(sum[i - 1][j - 1], j >= k ? sum[i - 1][j - k] : 0);
			int t = (i - 1) * (k - 1) + k - j;
			if(j <= m && j % (k - 1) == m % (k - 1) && t <= n && t % (k - 1) == n % (k - 1)) 
				ans = ADD(ans, f[i][j][1]);
			if(j) sum[i][j] = sum[i][j - 1];
			sum[i][j] = ADD(sum[i][j], ADD(f[i][j][0], f[i][j][1]));
		}
	printf("%d", ans);
	return 0;
}
```


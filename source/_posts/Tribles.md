---
title: Tribles
date: 2019-01-26 21:28:45
tags: [概率期望,dp]
---

# Description

给你$$k$$个球，  一个球可以活一天，在它死的时候会有概率$$pi$$生出$$i$$个小球$$(0<=i<n)$$， 现在问你$$m$$天后 所有小球全部死亡的概率是多少？

<!--more-->

# Soltuion

由于每个球是独立的，我们可以把每个答案算出来，最后k次方一下就好了；

问题就转变为开始有一个球，过m天后全部死亡的概率是多少？

我们设f[i]表示**还有i天所有球全部死亡的概率**，则有：
$$
f[i]=p0+p1*f[i-1]^1+p2*f[i-1]^2...
$$
为什么是$$f[i-1]$$呢？

因为今天每个球又变成了独立的个体，如果一个球生了$$j$$个，那么这$$j$$个必然在$$i-1$$天后死去，而这$$j$$个球也是独立的个体，所以每个球死亡的概率都是$$f[i-1]$$,所以要$$j$$次方。

最后，统计一下答案，就可以了。

# Code

```c++
#include<cstdio>
#include<cmath>
#include<cstring>
using namespace std;
const int maxN=1000 + 100;
double p[maxN+1],f[maxN+1];
int n,m,k,t,cnt;
int main()
{
	scanf("%d",&t);
	while(t--)
	{
		scanf("%d%d%d",&n,&k,&m);
		for(int i=0;i<n;i++) scanf("%lf",&p[i]);
		f[0]=0; f[1]=p[0];
		for(int i=2;i<=m;i++)
		{
			f[i]=0;
			for(int j=0;j<n;j++) f[i]+=p[j]*pow(f[i-1],j);
		}
		printf("Case #%d: %lf\n",++cnt,pow(f[m],k));
	}
	return 0;
}
```


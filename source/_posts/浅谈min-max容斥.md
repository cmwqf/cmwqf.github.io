---
title: 浅谈min-max容斥
date: 2019-07-25 15:46:37
tags: [组合数学]
---

# Min-max容斥

早就听闻Min-max容斥的大名，今天学习了一下，感觉妙不可言。

给定集合$S$，设$max(S)$为$S$中的最大值，$min(S)$为$S$中的最小值，那么我们可以得到：
$$
max(S)=\sum_{T\subset S}(-1)^{|T|-1}min(T)
$$
举个例子：
$$
max(a,b)=min(a)+min(b)-min(a,b)\\
max(a,b,c)=min(a)+min(b)+min(c)-min(a,b)-min(b,c)-min(a,c)+min(a,b,c)
$$
我们来看一下，假设$a<=b<=c$，则后面那个式子右边等于
$$
a+b+c-a-b-a+a=c=max(a,b,c)
$$
是不是感觉很神奇？

# K-th min-max容斥

我们计$max_k(S)$表示$S$中第$k$大的元素，那么我们有：
$$
max_k(S)=\sum_{T\sub S}(-1)^{|T|-k}C(|T|-1,k-1)min(T)
$$
具体的证明什么的，我当然看不懂了，记住就好了。

# 期望的min-max容斥

由于期望的线性性，$min-max$容斥适用于期望。
$$
E(max(S))=\sum_{T\sub S}(-1)^{|T|-1}E(min(T))\\
E(max_k(S))=\sum_{T\sub S}(-1)^{|T|-k}C(|T|-1,k-1)E(min(T))
$$

# 例题

[HAOI2015按位或]()
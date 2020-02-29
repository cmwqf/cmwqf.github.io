---
title: AGC038E Gachapon
date: 2020-02-29 12:34:17
tags: [dp, 概率期望, 容斥原理]
categories: [Atcoder]
---

# Description

由一个随机数生成器，每次生成一个$[0,n-1]$的数，每个数出现的概率为$\frac{a_i}{S}$（$S=\sum a_i$）。

我们现在要求每个数$i$都满足其出现次数$>=b_i$的期望时间。

其中$n\le 400,\sum a_i \le 400, \sum b_i \le 400$。

<!--more-->
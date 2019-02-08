---
title: Endless Spin
date: 2019-01-26 19:52:48
tags: [概率期望,dp,min-max容斥]
---

首先，我们要讲一下什么是min-max容斥：

给定集合S，设max(S)为集合中最大值，min(S)为集合中最小值，则有：
$$
max(S)=\sum_{T \subset S}(-1)^{|T|-1}min(T)
$$



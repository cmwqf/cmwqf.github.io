---
title: 浅谈FWT快速沃尔什变换
date: 2019-07-06 23:05:27
tags: [多项式]
---

# 快速沃尔什变换

FWT与FFT类似，只不过是进行集合卷积的计算，如：
$$
C_k=\sum_{i\oplus j = k} A_i*B_j\\
其中\oplus是位运算,可以是与,或,异或等
$$

# 或(or)运算的FWT

我们考虑或的FWT，如果我们有一个集合$A_i$，设$FWT(A)=A'$那么$A'_i$的意义就是这个集合所有子集的贡献之和。
$$
A'_i=\sum_{i|j=i}A_j
$$
这样，因为$k|(i|j)=k$可以推出$k|i=k,k|j=k$我们可以得到一个结论：
$$
C'_k=\sum_{k|t=k}C_t=\sum_{k|t=k}\sum_{i|j=t}A_i*B_j=\sum_{k|(i|j)=k}A_i*B_j\\
=(\sum_{k|i=k}A_i)*(\sum_{k|j=k}B_j)=A'_k*B'_k
$$
这样我们来看$A'$是怎么求的。
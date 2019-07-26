---
title: 浅谈FWT快速沃尔什变换
date: 2019-07-06 23:05:27
tags: [多项式,FWT]
---

# 快速沃尔什变换

FWT与FFT类似，只不过是进行集合卷积的计算，如：
$$
C_k=\sum_{i\oplus j = k} A_i*B_j\\
其中\oplus是位运算,可以是与,或,异或等
$$
<!--more-->

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

我们考虑吧$A$长度补为$2^k$，分为两半，$A0$是前面一半，表示第一位是0，$A1$是后面一半，表示第一位是1。

然后，我们也把$A'$分为两半来考虑，前面一半是第一位是0，那么它的子集就是$A0'$（它本身），后面一半第一位是1，它的子集既有$A1‘$(它本身)，还有$A0’$。

所以
$$
A'=merge(A0',A0'+A1')\\
即FWT(A)=merge(FWT(A_0),FWT(A_0)+FWT(A_1))
$$
其中，$merge$就是将两个数组拼接在一起，$+$表示对应下标的数相加。

这样我们就能在$O(nlogn)$的复杂度内求出了$FWT(A)$。

这样，我们就可以算出$FWT(C)$，那么，我们怎么把$FWT(C)$转回来呢？

$$
FWT(A)_0:FWT(A)的前半部分,FWT(A_0):A的前半部分的FWT\\
\because FWT(A)_0=FWT(A_0)\\
\therefore A_0 = IFWT(FWT(A_0))=IFWT(FWT(A)_0)\\
\because FWT(A)_1=FWT(A_0)+FWT(A_1)\\
\therefore A_1=IFWT(FWT(A_1))=IFWT(FWT(A)_1-FWT(A)_0)
$$

## 结论

$$
FWT(A)=
\begin{cases}
(FWT(A_0),FWT(A_0+A_1))\quad(n\neq 1)\\
A\quad(n=1)
\end{cases}
$$

下面写$IFWT$，注意，此时大括号后面的$A$是已经$FWT$过后的$A$
$$
IFWT(A)=
\begin{cases}
(IFWT(A_0),IFWT(A_1-A_0))\quad(n \neq 1)\\
A\quad(n = 1)
\end{cases}
$$

## Code

```c++
inline void FWT(int *a, int type)
{
    for(int mid = 1; mid < lim; mid <<= 1)
        	for(int i = 0; i < lim; i += (mid << 1))
                	for(int j = 0; j < mid; j++)
                        a[i + mid + j] += a[i + j] * type;
}
```

# 与(and)运算的FWT

证明太麻烦了，直接上结论吧，这个和上面或的差不多。

## 结论

$$
FWT(A)=
\begin{cases}
(FWT(A_0+A_1),FWT(A_1))\quad(n\neq 1)\\
A\quad (n=1)
\end{cases}\\
IFWT(A)=
\begin{cases}
(IFWT(A_0-A_1),IFWT(A_1))\quad (n\neq 1)\\
A\quad(n=1)
\end{cases}
$$

## Code

```c++
inline void FWT(int *a, int type)
{
	for(int mid = 1; mid < lim; mid <<= 1)
		for(int i = 0; i < lim; i += (mid << 1))
			for(int j = 0; j < mid; j++)
				a[i + j] += a[i + mid + j] * type;
}
```



# 异或(xor)运算的FWT

这个最难。

尝试证明，却失败.jpg

## 结论

$$
FWT(A)=
\begin{cases}
(FWT(A_0)+FWT(A_1),FWT(A_0)-FWT(A_1))\quad (n \neq 1)\\
A\quad (n = 1)
\end{cases}\\
IFWT(A)=
\begin{cases}
(\frac{IFWT(A_0)+IFWT(A_1)}{2},\frac{IFWT(A_0)-IFWT(A_1)}{2})\quad(n\neq 1)\\
A\quad(n=1)
\end{cases}
$$
## Code

```c++
inline void FWT(int *a, int type)
{
    for(int mid = 1; mid < lim; mid <<= 1)
        for(int i = 0; i < lim; i += (mid << 1))
            for(int j = 0; j < mid; j++)
            {
                int x = a[i + j], y = a[i + mid + j];
                a[i + j] = x + y;
                a[i + mid + j] = x - y;
                if(type == -1) a[i + j] /= 2, a[i + mid + j] /= 2;
            }
}
```






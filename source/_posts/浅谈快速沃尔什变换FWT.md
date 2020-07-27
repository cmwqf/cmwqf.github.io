---
title: 浅谈快速沃尔什变换FWT
date: 2019-07-06 23:05:27
tags: [多项式,FWT]
categories: [学习笔记]
---

# 快速沃尔什变换

其实$$FWT$$与$$FFT$$类似，只不过是进行集合卷积的计算，如：
$$
C_k=\sum_{i\oplus j = k} A_i*B_j\\
其中\oplus是位运算,可以是与,或,异或等
$$
<!--more-->

之前看到一个形象的比喻，所谓这些变换，就是相当于你要过一条马路，但是直接过不好走，那么我们要上一个天桥（正变换），然后从天桥上走过去，最后再走下来（逆变换）。

# 或(or)运算的FWT

我们考虑或的$$FWT$$，如果我们有一个集合$$A_i$$，设$$FWT(A)=A'$$那么$$A'_i$$的意义就是这个集合所有子集的贡献之和。
$$
A'_i=\sum_{i|j=i}A_j
$$
这样，因为$$k|(i|j)=k$$可以推出$$k|i=k,k|j=k$$我们可以得到一个结论：
$$
C'_k=\sum_{k|t=k}C_t=\sum_{k|t=k}\sum_{i|j=t}A_i*B_j=\sum_{k|(i|j)=k}A_i*B_j\\
=(\sum_{k|i=k}A_i)*(\sum_{k|j=k}B_j)=A'_k*B'_k
$$
这样我们来看$$A'$$是怎么求的。

我们考虑吧$$A$$长度补为$$2^k$$，分为两半，$$A0$$是前面一半，表示第一位是0，$$A1$$是后面一半，表示第一位是1。

然后，我们也把$$A'$$分为两半来考虑，前面一半是第一位是0，那么它的子集就是$$A0'$$（它本身），后面一半第一位是1，它的子集既有$$A1'$$(它本身)，还有$$A0’$$。

所以
$$
A'=merge(A0',A0'+A1')\\
即FWT(A)=merge(FWT(A_0),FWT(A_0)+FWT(A_1))
$$
其中，$$merge$$就是将两个数组拼接在一起，$$+$$表示对应下标的数相加。

这样我们就能在$$O(nlogn)$$的复杂度内求出了$$FWT(A)$$。

这样，我们就可以算出$$FWT(C)$$，那么，我们怎么把$$FWT(C)$$转回来呢？

$$
FWT(A)_0:FWT(A)的前半部分,FWT(A_0):A的前半部分的FWT\\
\because FWT(A)_0=FWT(A_0)\\
\therefore A_0 = IFWT(FWT(A_0))=IFWT(FWT(A)_0)\\
\because FWT(A)_1=FWT(A_0)+FWT(A_1)\\
\therefore A_1=IFWT(FWT(A_1))=IFWT(FWT(A)_1-FWT(A)_0)
$$

实际上，写那么多式子令人烦躁，考虑因为是$$or$$运算，所以递归每一位的时候，只有当前这一位是$$0$$的对后面相对应的这一位是$$1$$的产生贡献，然后逆变换就是反过来。

但是这个时候就会有个疑惑，正变换是先处理左右两边再算贡献，那么逆变换要把过程反过来，不应该先算贡献再递归左右两边吗，为什么还是一个函数？

实际上，因为考虑正变换时每一位的顺序是无所谓的，我们按位正着算和倒着算是等价的。

所以我们逆变换便假设正变换是倒着枚举每一位来计算的，那么先递归左右两边再计算贡献就是合理的了，这个要仔细思考。

## 结论

$$
FWT(A)=
\begin{cases}
(FWT(A_0),FWT(A_0+A_1))\quad(n\neq 1)\\
A\quad(n=1)
\end{cases}
$$

下面写$$IFWT$$，注意，此时大括号后面的$$A$$是已经$$FWT$$过后的$$A$$
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

证明太麻烦了，直接上结论吧，这个和上面或的差不多，与的话就是正好反过来，让这一位是$$1$$的对这一位是$$0$$的进行贡献。

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

这个比较难。

与前面一样的想法，我们考虑构造一个序列，但这个序列比较奇怪。

我们定义$$x\otimes y$$为$$x\& y$$之后的位数的奇偶性（奇数为$$1$$，偶数为$$0$$）。

这个运算满足$$(i\otimes j)\oplus (i\otimes k)=i\otimes (j\oplus k)$$（有点像乘法分配律），可以用来进行异或卷积。

我们构造
$$
A_n=\sum_{n\otimes i=0}a_i-\sum_{n\otimes i=1}a_i
$$
这个看起来很奇怪，但是
$$
C_n=\sum_{n\otimes k=0}c_k-\sum_{n\otimes k=1}c_k\\
=\sum_{n\otimes k=0}\sum_{i\oplus j=k}a_ib_j-\sum_{n\otimes k=1}\sum_{i\oplus j=k}a_ib_j\\
=\sum_{n\otimes (i\oplus j)=0}a_ib_j-\sum_{n\otimes (i\oplus j)=1}a_ib_j\\
=(\sum_{n\otimes i=0}a_i)*(\sum_{n\otimes j=0}b_j)+(\sum_{n\otimes i=1}a_i)*(\sum_{n\otimes j=1}b_j)\\-(\sum_{n\otimes i=0}a_i)*(\sum_{n\otimes j=1}b_j)-(\sum_{n\otimes i=1}a_i)*(\sum_{n\otimes j=0}b_j)\\
=(\sum_{n\otimes i=0}a_i-\sum_{n\otimes i=1}a_j)*(\sum_{n\otimes i=0}a_i-\sum_{n\otimes i=1}a_j)\\
=A_n*B_n
$$
那么，这个怎么$$FWT$$呢？

正变换，依然考虑分治，分为$$A_0$$，$$A_1$$，表示当前这一位（$$i$$）是$$0/1$$，这个时候，我们考虑$$j$$和$$j+2^i$$，（$$j$$的第$$i$$位为$$0$$），我们设$$[i,j]$$表示$$i$$这一层$$j$$的值，那么有如下几种转移（考虑位数中$$1$$的变化）：

第一，$$[i-1,j]->[i,j]$$位数增加的为$$0\& 0=0$$，所以贡献为$$+1$$。

第二，$$[i-1,j+2^i]->[i,j]$$增加的为$$1\& 0=0$$，贡献也为$$+1$$。

第三，$$[i-1,j]->[i,j+2^i]$$增加的为$$0\& 1=0$$，贡献是$$+1$$。

第四，$$[i-1,j+2^i]->[i,j+2^i]$$增加的为$$1\& 1=1$$，位数中$$1$$奇偶性发生变化，符号取反，贡献为$$-1$$。

因此，就有
$$
A=merge(A_0+A_1,A_0-A_1)
$$
那么逆变换也就很简单了，
$$
c=merge(\frac{c_0+c_1}{2},\frac{c_0-c_1}{2})
$$

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

# 子集卷积

在一些题目中，有的时候要求
$$
c[S]=\sum_{T\subset S}a[T]*b[S\oplus T]
$$
这个时候应该怎么办呢？

我们考虑如果$$i,j$$满足$$j=k\oplus i$$的话，当且仅当$$i|j=k, |i|+|j|=|k|$$，那么我们就是要求
$$
c[k]=\sum_{i|j=k}[|i|+|j|=|k|]*a[i]*b[j]
$$
那么，我们多记一维，设$$a[t][k]$$表示$$k$$中有$$t$$个$$1$$。

每次就是要
$$
c[s+t][i|j]+=a[s][i]*b[t][j]
$$
最后只要统计所有$$c[s][i]$$（$$i$$的位数为$$s$$）作为$$ans[i]$$的答案即可。

那么，我们应该怎么算这个东西呢？

我们把第二维都正向地$$FWT$$一下，然后枚举位数，暴力卷积（可以证明这是对的），具体见代码：

```c++
inline void work()
{
    for(int i = 0; i <= 20; i++) FWTOR(C[i], 1);
    for(int i = 0; i <= 20; i++)//枚举位数
    {
        memset(D, 0, sizeof(D));
        for(int j = 0; j <= i; j++)
            for(int s = 0; s < (1 << 20); s++)
                D[s] = ADD(D[s], 1ll * C[j][s] * C[i - j][s] % mod);
        FWTOR(D, -1);
        for(int s = 0; s < (1 << 20); s++)
            if(bc[s] == i) ans[s] = ADD(ans[s], D[s]);
    }
}
```






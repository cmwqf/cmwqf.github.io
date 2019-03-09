---
title: 分治FFT详解
date: 2019-02-18 16:40:57
tags: [多项式]
---

# 什么是分治FFT

分治FFT用于快速计算这样一类式子：
$$
f[n]=\sum_{i=0}^{n-1}f[i]g[n-i]
$$
与普通的FFT不同，这个式子是用自己来更新自己，所以如果暴力用FFT，时间复杂度为$$O(n^2logn)$$，因此，我们考虑用分治FFT来解决这类问题。

<!--more-->

# 分治FFT的实现

分治，指的就是cdq分治（运用的是这种思想）：

就是我们考虑先计算出$$f[l],f[l+1],...,f[mid]$$的值，再计算它对$$f[mid+1],...,f[r]$$的贡献，最终完成计算。

由于计算l~mid的值是递归的事情，所以我们只要考虑$$f[l...mid]$$对于$$f[mid+1...r]$$的贡献即可。

首先，我们考虑，在$$[mid+1,r]$$的每一个数$x$,$$[l,mid]$$区间对其贡献为：
$$
w[x]=\sum_{i=l}^{mid}f[i]g[x-i]
$$
我们将求和的范围扩大到$$[l,x-1]$$，因为$$f[mid+1]->f[x-1]$$都是0，所以对式子的值并无影响。
$$
w[x]=\sum_{i=l}^{x-1}f[i]g[x-i]
$$
这时，我们考虑计算右边那个式子，我们可以设$$a[i]=f[i+l],b[i]=g[i+1]$$

这个式子就变化为
$$
\sum_{i=0}^{x-l-1}a[i]b[x-l-1-i]
$$
由卷积的知识可以得知，这个式子可以看成
$$
c[x-l-1]=\sum_{i=0}^{x-l-1}a[i]b[x-l-1-i]
$$
这个式子，就可以用FFT来完成了。结合上面那个$$w[x]$$的式子，可以得到：
$$
w[x]=c[x-l-1]=\sum_{i=0}^{x-l-1}a[i]b[x-l-1-i]
$$
所以，我们在做完卷积之后，直接：
$$
w[x]=a[x-l-1](a为卷积后的数组)
$$
最后，将贡献加上去就可以了：
$$
f[x]+=w[x]，即f[x]+=a[x-l-1]
$$
因此，我们所需要计算的数组的长度为$$r-l-1$$，时间复杂度为$$O(nlog^2n)$$

# 模板题与代码

题目：[分治FFT](https://www.luogu.org/problemnew/show/P4721)

代码：

```c++
#include<cstdio>
#include<iostream>
#define LL long long
using namespace std;
const LL maxN=5e6 + 100,G=3,Gi=332748118,mod=998244353;
LL a[maxN+1],b[maxN+1];
LL f[maxN+1],g[maxN+1];
int n,m;
int R[maxN+1];
inline LL read()
{
	int num=0,f=1;
	char ch=getchar();
	while(!isdigit(ch)) {if(ch=='-') f=-1; ch=getchar();}
	while(isdigit(ch)) num=(num<<3)+(num<<1)+(ch^48),ch=getchar();
	return num*f;
}
inline LL pow(LL a,LL x)
{
	LL ans=1;
	while(x)
	{
		if(x&1) ans=ans*a%mod;
		a=a*a%mod;
		x>>=1;
	}
	return ans;
}
inline void NTT(LL *a,int type,LL lim)
{
	for(int i=0;i<lim;i++)
	   if(i<R[i]) swap(a[i],a[R[i]]);
	for(int mid=1;mid<lim;mid<<=1)
	{
		LL wn=pow(type==1 ? G : Gi,(mod-1)/(mid<<1));
		for(int i=0;i<lim;i+=(mid<<1))
		{
			LL w=1;
			for(int j=0;j<mid;j++,w=w*wn%mod)
			{
				int x=a[i+j],y=a[i+mid+j]*w%mod;
				a[i+j]=(x+y)%mod,a[i+mid+j]=(x-y+mod)%mod;
			}
		}
	}
}
inline void calc(LL lim)
{
	NTT(a,1,lim); NTT(b,1,lim);
	for(int i=0;i<lim;i++) a[i]=a[i]*b[i]%mod;
	NTT(a,-1,lim);
	LL inv=pow(lim,mod-2);
	for(int i=0;i<lim;i++) a[i]=a[i]*inv%mod;
}
inline void cdqFFT(int l,int r)
{
	if(l==r) return;
	int mid=(l+r)>>1;
	cdqFFT(l,mid);
	LL lim=1,cnt=0;
	while(lim<=(r-l-1)) lim<<=1,cnt++;
	for(int i=0;i<lim;i++) R[i]=(R[i>>1]>>1) | ((i&1) << cnt-1);
	for(int i=0;i<lim;i++) a[i]=b[i]=0;
	for(int i=0;i<=mid-l;i++) a[i]=f[i+l];
	for(int i=0;i<=r-l-1;i++) b[i]=g[i+1];
	calc(lim);
	for(int i=mid+1;i<=r;i++) f[i]=(f[i]+a[i-l-1])%mod;
	cdqFFT(mid+1,r);
} 
int main()
{
	n=read(); f[0]=1;
	for(int i=1;i<n;i++) g[i]=read();
	cdqFFT(0,n-1); 
	for(int i=0;i<n;i++) printf("%lld ",f[i]);
	return 0;
}
```






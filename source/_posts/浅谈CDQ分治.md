---
title: 浅谈CDQ分治
date: 2019-01-13 19:31:19
tags: [CDQ分治]
categories: [学习笔记]
---

# CDQ分治

这是来解决三维偏序的题的，那么什么是三维偏序呢？

具体来说，就是有n个三元组$$(a,b,c)$$,求
$$
f(i)=(a_j<=a_i \&\&b_j<=b_i\&\&c_j<=c_i)
$$
的$$j$$的个数。

<!--more-->

# 原理

首先，我们考虑先把a从小到大来排序，这样，在后面的操作中，就不用再考虑a的大小关系了；

然后呢？ 我们的CDQ分治就上场了

对于一个子序列$$(l,r)$$，我们把它分为$$(l,mid)$$和$$(mid+1,r)$$两个子序列分治，在每一个分治中把b从小到大排好序,就是这样

```
void cdq(int l,int r)
{
	if(l==r) ...(统计单点贡献) return;
    int mid=(l+r)>>1;
    cdq(l,mid); cdq(mid+1,r);
}
```
考虑此时，$$(l,mid)$$和$$(mid+1,r)$$都其内部的b已经是有序的了；

那么有人问：b是有序的，那a不就变成乱序的了吗？

对啊，a的确变成乱序的了，但这只是在$$(l,mid)$$和$$(mid+1,r)$$中是乱序的，但就现在要操作的序列$$(l,r)$$来看，$$(l,mid)$$中的所有的a都是小于$$(mid+1,r)$$中的a的；

而cdq的操作，就是来计算$$(l,mid)$$对$$(mid+1,r)$$的贡献，所以并不影响统计

下面，我们就对$$(l,r)$$进行统计：

考虑归并排序，一个指针 i 指向l,一个指针 j 指向mid+1，再用一个tmp数组来存临时；

当$$b[i]<=b[j]$$时，说明$$i$$对$$j$$可能有贡献（因为还不知道c的情况），就先把它放到一个树状数组里；

这个树状数组又是干什么的呢？

我们考虑二维偏序时,先将$$a$$排序，然后
```
for(auto b)
	ans+=query(b),change(b,1);
```

这样，每次动态维护这个树状数组，就能求出$$b[i]<=b[j]$$的个数了；

三维偏序也是一个想法，对于$$(mid+1,r)$$中的某一个$$j(mid+1<=j<=r)$$来说，由于$$(l,mid)$$中的b是排好序的，所以只要找到一个$$b[i]>b[j]$$的，那么对于后面的$$i$$都必定对$$j$$产生不了贡献；

这个时候就可以统计$$(l,mid)$$中对$$j$$的贡献了，并把$$j$$的指针下移了；

怎么统计贡献呢，很简单，树状数组里不是已经存了$$(l,i-1)$$中的c的值了吗，只要查询在树状数组里有多少个值$$<=c[j]$$的就可以了，

```
int i=l,j=mid+1,tot=l;//tot:临时数组的指针
while(i<=mid&&j<=r)
	if(b[i]<=b[j]) change(c[i],cnt[i]),...(copy),i++;//cnt[i]后面再解释
    else ans[j]+=query(c[j]),...(copy),j++;
```

最后像归并排序那样，但是要注意先移动$$j$$,因为有可能$$j$$还没有被统计完，要继续统计，再移动$$i$$，最后把原数组赋值成临时数组就好了；

注意，树状数组用完是要清空的，但是不能用$$memset$$,这样就T飞了，我是暴力一个个清空的。

大体的过程基本上就是这样，但是还是有很多细节要注意的：

首先，CDQ不能处理重复的三元组，所以在排序的时候要去重，再用$$cnt[i]$$来保存第i个三元组出现的次数，这就是上面留的坑，所以每次统计的时候要把$$cnt[i]$$加入树状数组；

同时，相同的对自己本身也有贡献，这就是当$$l==r$$时要统计一下自身贡献的原因；

还有，就是排序的时候要注意，不能只排$$a$$,要多关键字排序

```
bool comp(Node a,Node b)
{
	return a.a<b.a || (a.a==b.a&&a.b<b.b) || (a.a==b.a&&a.b==b.b&&a.c<b.c);
}
```
然后，基本上就没什么了吧

# Code

```c++
#include<cstdio>
#include<iostream>
#include<algorithm>
using namespace std;
const int maxN=1e5 + 100;
int n,k,res;
struct Node
{
	int a,b,c,cnt,ans;
}num[maxN*2+1],tmp[maxN*2+1];
int C[maxN*4+1],ans[maxN*2+1];
int read()
{
	int num=0,f=1;
	char ch=getchar();
	while(!isdigit(ch)) {if(ch=='-') f=-1;ch=getchar();}
	while(isdigit(ch)) num=(num<<3)+(num<<1)+(ch^48),ch=getchar();
	return num*f;
}
inline int lowbit(int x) {return x&(-x);}
inline void change(int x,int num)
{
	for(int i=x;i<=k;i+=lowbit(i)) C[i]+=num;
}
inline int query(int x)
{
	int ans=0;
	for(int i=x;i>0;i^=lowbit(i)) ans+=C[i];
	return ans;
}
bool comp(Node a,Node b)
{
	return a.a<b.a || (a.a==b.a&&a.b<b.b) || (a.a==b.a&&a.b==b.b&&a.c<b.c);
}
inline void cdq(int l,int r)
{
	if(l==r) { num[l].ans+=num[l].cnt-1; return;}
	int mid=(l+r)>>1;
	cdq(l,mid); cdq(mid+1,r);
	int i=l,j=mid+1,tot=l;
	while(i<=mid&&j<=r)
		if(num[i].b<=num[j].b) change(num[i].c,num[i].cnt),tmp[tot++]=num[i++];
		else num[j].ans+=query(num[j].c),tmp[tot++]=num[j++];
	while(j<=r) num[j].ans+=query(num[j].c),tmp[tot++]=num[j++];
	for(int k=l;k<i;k++) change(num[k].c,-num[k].cnt);
	while(i<=mid) tmp[tot++]=num[i++];
	for(int k=l;k<=r;k++) num[k]=tmp[k];
}
int main()
{
	n=read(),k=read();
	for(int i=1;i<=n;i++) num[i].a=read(),num[i].b=read(),num[i].c=read(),num[i].cnt=1;
	sort(num+1,num+n+1,comp);
	for(int i=1;i<=n;i++)
	{
		if(i==1 || !(num[i].a==num[i-1].a&&num[i].b==num[i-1].b&&num[i].c==num[i-1].c)) num[++res]=num[i];
		else num[res].cnt++;
	}
	cdq(1,res);
	for(int i=1;i<=res;i++) ans[num[i].ans]+=num[i].cnt;
	for(int i=0;i<n;i++) printf("%d\n",ans[i]);
	return 0;
}
```
---
title: K-D Tree进阶
date: 2019-02-04 15:12:49
tags: [K-D Tree]
categories: [学习笔记]
---

# 新的KD-Tree写法

最近做了一些题目后，发现之前写的模板不好记，且难以用于矩形数点（可能使我太弱了），但是，我在zzq等大佬的博客中找到了另一种写法，这里以二维的写法为标准（因为我发现大部分题目都是在二维上进行操作）：

<!--more-->

## 节点的存储

这种写法，tree节点存的是

```c++
struct KD
{
    int l,r;//左右子树
}tree[maxN+1];
struct Node
{
    int p[2];
}lu[maxN+1],rd[maxN+1],cur[maxN+1];
//lu:当前这个节点所管辖的矩形的左上角，ru:当前这个节点所管的矩形的右下角
//cur:当前这个节点所代表的点
```

具体来说，lu.x就是它所管的所有点x的最小值，lu.y就是y的最小值，而rd是最大值；

为什么要记录它所管的矩形呢？

这样，我们在矩形数点的时候，我们就可以像线段树一样查看此矩形是否在所查询的矩形之内，而在求平面k近点时，好像并没有什么特别的用处，但是在求平面k远点的时候，可以根据要查询的点到此矩形四个顶点的距离来剪枝，计算先从哪个地方出发。

## 建树

关于建树，我们可以用动态开点，来记录每一个节点的l和r。

一般题目都是二维上做文章，所以我们无需通过找方差来建树，可以直接以一层横线，一层竖线来划分，即

```c++
inline void build(int l,int r,int d)
{
    ...
    build(l,mid-1,d^1); build(mid+1,r,d^1);
}
```

这样好写，效果也达到了。

我们在nth_element过后，要记录当前树的节点是哪个点，并且记录左子树和右子树，最后要更新一下节点，计算它的lu,rd的值，如果是矩形数点的话，还要更新tree[node]中的num值。

代码如下：

```c++
inline int build(int l,int r,int flag)
{
	if(l>r) return 0;
	int mid=(l+r)>>1,node=++cnt;
	now=flag;
	nth_element(a+l,a+mid,a+r+1,comp);
	lu[node]=rd[node]=cur[node]=a[mid];
	tree[node].num=1ll*a[mid].value;
	tree[node].l=build(l,mid-1,flag^1);
	tree[node].r=build(mid+1,r,flag^1);
	pushup(node);
	return node; 
}
```

## 查询

### 矩形数点

我们来讲一讲矩形数点时的查询：

1.我们要看这个节点所代表的矩形是否在要查询的矩形之外，如果完全在外面，就直接返回；

2.我们要看这个节点所代表的矩形是否全部被包含在要查询的矩形之内，如果完全被包含，直接统计答案，返回；

3.我们要看这个节点所代表的点是否在矩形内（因为再往左右子树找是不包含此点的），如果是，统计答案，但是**不能返回**；

4.往左右两个子树查找。

至于是否包含这类，就通过比较要查询的矩形x,y和当前矩形的lu,rd就可以了，代码如下：

```c++
inline bool fullout(int node)
{
	for(int i=0;i<K;i++) if(mx.p[i]<lu[node].p[i]||rd[node].p[i]<mn.p[i]) return true;
	return false;
}
inline bool fullin(int node)
{
	for(int i=0;i<K;i++) if(!(mx.p[i]>=rd[node].p[i]&&mn.p[i]<=lu[node].p[i])) return false;
	return true;
}
inline bool pointin(int node)
{
	for(int i=0;i<K;i++) if(!(cur[node].p[i]<=mx.p[i]&&cur[node].p[i]>=mn.p[i])) return false;
	return true;
}
inline void query(int node)
{
	if(fullout(node)) return;//全部在外 
	if(fullin(node)) {ans+=tree[node].num; return;}//全部在内 
	if(pointin(node)) ans+=cur[node].value;//单点在内 
	query(tree[node].l); query(tree[node].r);
}
```

### 最近点

下面给出二维k-d树寻找距离点P最近的点：

1.设定初始答案ans=inf

2.从根节点开始，先用根代表的节点更新答案，由于左右儿子都可能存在最近点，我们优先选择距离P最近的矩形递归查询

3.以P为圆心，ans为半径画圆，若与之前未递归的矩形相交，就再往那查询；

具体做法与上一篇[浅谈K-D Tree](https://cmwqf.github.io/2019/02/03/%E6%B5%85%E8%B0%88K-D-Tree/)相近，就不给出代码了

### k远点

我们维护一个小根堆，初始向里面放入k个-inf，然后之前与ans比较就变成了与对顶比较了。

（先留个坑，代码以后再写）

## 插入

对，K-D Tree还能插入，但是插入之后，复杂度便不能保证了，但是插入还是很既简单的，就是比较K-D Tree与当前节点在d维度上的值，来决定到底是往左还是往右，但是要记得更新当前节点。

（代码也以后再补吧）

## 模板&例题

[CQOI2017老C的任务](https://www.luogu.org/problemnew/show/P3755)

是一道省选题，出题人应该不想让K-D Tree过掉，但是我开了O2还是水过去了。

最后一次提交是把O2去掉之后的结果。。。

就是一个矩形数点的模板题，代码：

```c++
#include<cstdio>
#include<cmath>
#include<iostream>
#include<algorithm>
#define ls tree[node].l
#define rs tree[node].r
using namespace std;
const int maxN=1e5 + 100,K=2,inf=1e9;
struct Node
{
	int p[2],value;
}a[maxN+1];
struct KD
{
	int l,r;
	long long num;
}tree[maxN+1];
int n,m,now,cnt;
int root;
long long ans;
Node lu[maxN+1],rd[maxN+1],cur[maxN+1];
Node mn,mx;
inline int read()
{
	int num=0,f=1;
	char ch=getchar();
	while(!isdigit(ch)) {if(ch=='-') f=-1; ch=getchar();}
	while(isdigit(ch)) num=(num<<3)+(num<<1)+(ch^48),ch=getchar();
	return num*f;
}
inline bool comp(Node a,Node b) {return a.p[now]<b.p[now];}
inline void pushup(int node)
{
	tree[node].num+=tree[ls].num+tree[rs].num;
	for(int i=0;i<K;i++) 
	{
		lu[node].p[i]=min(lu[node].p[i],min(lu[ls].p[i],lu[rs].p[i]));
		rd[node].p[i]=max(rd[node].p[i],max(rd[ls].p[i],rd[rs].p[i]));
	}
}
inline int build(int l,int r,int flag)
{
	if(l>r) return 0;
	int mid=(l+r)>>1,node=++cnt;
	now=flag;
	nth_element(a+l,a+mid,a+r+1,comp);
	lu[node]=rd[node]=cur[node]=a[mid];
	tree[node].num=1ll*a[mid].value;
	tree[node].l=build(l,mid-1,flag^1);
	tree[node].r=build(mid+1,r,flag^1);
	pushup(node);
	return node; 
}
inline bool fullout(int node)
{
	for(int i=0;i<K;i++) if(mx.p[i]<lu[node].p[i]||rd[node].p[i]<mn.p[i]) return true;
	return false;
}
inline bool fullin(int node)
{
	for(int i=0;i<K;i++) if(!(mx.p[i]>=rd[node].p[i]&&mn.p[i]<=lu[node].p[i])) return false;
	return true;
}
inline bool pointin(int node)
{
	for(int i=0;i<K;i++) if(!(cur[node].p[i]<=mx.p[i]&&cur[node].p[i]>=mn.p[i])) return false;
	return true;
}
inline void query(int node)
{
	if(fullout(node)) return;//全部在外 
	if(fullin(node)) {ans+=tree[node].num; return;}//全部在内 
	if(pointin(node)) ans+=cur[node].value;//单点在内 
	query(tree[node].l); query(tree[node].r);
}
int main()
{
	n=read(),m=read();
	for(int i=1;i<=n;i++)
		a[i].p[0]=read(),a[i].p[1]=read(),a[i].value=read();
	lu[0].p[0]=lu[0].p[1]=inf; rd[0].p[0]=rd[0].p[1]=-inf;
	root=build(1,n,0);
	while(m--)
	{
		mn.p[0]=read(),mn.p[1]=read(),mx.p[0]=read(),mx.p[1]=read();
		ans=0; query(root);
		printf("%lld\n",ans);
	}
	return 0;
} 
```




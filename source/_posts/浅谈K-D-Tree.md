---
title: 浅谈K-D Tree
date: 2019-02-03 09:53:10
tags: [K-D Tree]
categories: [学习笔记]
---

# 什么是K-D Tree?

K-D Tree是一类玄学的暴力数据结构（有的时候复杂度玄学），但是，在随机情况下，它跑的非常快，是骗分的好方法（逃；

K-D Tree一般可以解决多维数点，最近点对，k近点对等问题，还是非常实用和高效的，它的单次查询时间复杂度为
$$
O(kn^{1-\frac{1}{k}}+logn)
$$
其中k是维数，如果是1维，就是logn（其实就是线段树），二维就是$$n\sqrt{n}$$。

<!--more-->

# K-D Tree的构造

一个数据结构之所以高效，就在于它的构造，K-D Tree的高效，就在于**把点按照某种特定的规则建成一种二叉树**

什么规则呢？

对于这棵树上的一个节点，相当于一个超平面，把空间分割成两块，其中一边，是这个节点的左儿子，另一边，则是它的右儿子，满足对于划分的依据维度第i维，使得左儿子的p[i]<p[mid]<右儿子的p[i]；

具体地说，对于当前区间[l,r]（这里的区间指点的序号区间，不是坐标区间），我们算出这些点每一维度的方差，取方差最大的那一维，设为d，作为我们的分裂方式，然后，我们将整个区间按照d维从小到大排序，取中间的那个点mid为分裂点，再以[l,mid-1]为左子树建树，[mid+1,r]为右子树建树。

这个建树过程，是严格的$$O(nlogn)$$。注意，我们每次找中位数的时候，可以不用排序，C++中的STL有一个叫nth_element的函数，具体用法如下：

```c++
#include<algorithm>
如果我们要查[l,r]中第k大的数：
nth_element(a+l,a+k,a+r+1,comp);//comp:比较方式
```

它可以保证第k大的数在a[k]的位置上，但是整个区间并不是有序的，可以保证k左边的都小于k，右边的都大于k，复杂度为$$O(r-l+1)$$；

这样，这棵K-D Tree就建好了。

# K-D Tree的查询

建好了树，我们怎么查询呢？其实，查询就是一个加了剪枝的暴力。

查询的时候，假设我们要查的点为op，查到了区间[l,r]，其根节点为mid，那么，我们先算出op到mid的距离，并更新答案，假设[l,r]是从d维划分的，那么，如果op的第d维的坐标小于mid第d维的坐标，就往mid的左子树找，但是，并不一定在左子树，找完以后还要判断一下，如果以op为圆心，当前的答案ans为半径的话，会不会与mid点的右半边有交，如果有，就还要往右边查询一下，因为有可能最近点在右子树，说白了，K-D Tree就是变化一下查询顺序，用当前的答案来剪枝，所以，只要出题人用点心，就可以轻松地把你卡到单次询问$$O(n)$$（在k近点对的时候），普通的最近点对最坏复杂度好像是$$O(n\sqrt{n})$$，但是，如果你想不到其他方法，这也是一个骗高分的好方法啊，查询的部分代码：

```c++
int cur=split[mid];//split[mid]为[l,r]的划分方式
int radius=(op.p[cur]-a[mid].p[cur])*(op.p[cur]-a[mid].p[cur]);
if(op.p[cur]<a[mid].p[cur])
{
    query(l,mid-1);
    if(radius<=ans) query(mid+1,r);//radius就是op到右半平面的距离
}else ...//同理
```

以上，就是K-D Tree的基本内容。

# 例题

[hdu2966](http://acm.hdu.edu.cn/showproblem.php?pid=2966)

平面最近点对的裸题，但是还是有要注意的地方，例如，它并不是给你一个点，让你找它的最近点，而是让你找这n个点中每个点的最近点，因此，必须要把本身给排除掉。

代码我就不给了，根据后面的模板随便改一下就可以了。

# Code

具体的看注释吧:

------

```c++
#include<cstdio>
#include<algorithm>
#include<cstring>
#include<iostream>
#define int long long
using namespace std;
const int maxN=1e5 + 100,K=2;//K的值根据情况改动
struct Node
{
    int p[10],id;    
}a[maxN*2+1],op,point,tmp[maxN+1];
int now,split[maxN+1],n;
int ans,k,t;
bool use[maxN+1];
double var[10];
inline int read()
{
    int num=0,f=1;
    char ch=getchar();
    while(!isdigit(ch)) {if(ch=='-') f=-1; ch=getchar();}
    while(isdigit(ch)) num=(num<<3)+(num<<1)+(ch^48),ch=getchar();
    return num*f;
}
inline bool comp(Node a,Node b) {return a.p[now]<b.p[now];}
inline void build(int l,int r)
{
    if(l>r) return;
    int mid=(l+r)>>1; now=0;
    for(int pos=1;pos<=K;pos++)
    {
        double ave=0.0; var[pos]=0.0;
        for(int i=l;i<=r;i++) ave+=a[i].p[pos];
        ave/=(r-l+1);//计算平均数
        for(int i=l;i<=r;i++) var[pos]+=(a[i].p[pos]-ave)*(a[i].p[pos]-ave);//方差
        if(!now||var[pos]>var[now]) now=pos;//更新划分方案
    }
    split[mid]=now;//划分方案
    nth_element(a+l,a+mid,a+r+1,comp);//中位数
    build(l,mid-1);
    build(mid+1,r);
}
inline void query(int l,int r)
{
    if(l>r) return;
    int mid=(l+r)>>1;
    int dis=0;
    for(int i=1;i<=K;i++) dis+=(op.p[i]-a[mid].p[i])*(op.p[i]-a[mid].p[i]);
    if(!use[a[mid].id]&&dis<ans)//更新答案
    {
        ans=dis; 
        point=a[mid];
    }
    int cur=split[mid];
    int radius=(op.p[cur]-a[mid].p[cur])*(op.p[cur]-a[mid].p[cur]);//计算op到分裂平面的距离
    if(op.p[cur]<a[mid].p[cur])
    {
        query(l,mid-1);
        if(radius<=ans) query(mid+1,r);
    }
    else
    {
        query(mid+1,r);
        if(radius<=ans) query(l,mid-1);
    }
}
#undef int
int main()
#define int long long
{
    k=1;//k指的是k近点
    t=read();
    while(t--)
    {
        n=read();
        for(int i=1;i<=n;i++)
        {
            for(int j=1;j<=K;j++) a[i].p[j]=read();//K指的是维度大小
            a[i].id=i; tmp[i]=a[i];
        }
        build(1,n);
        for(int i=1;i<=n;i++)
        {
            memset(use,false,sizeof(use));//判断k近点的时候，要把已经找到的标记，防止重复
            op=tmp[i]; use[i]=1;//将自己设为1（仅限于hdu这一题）
            for(int j=1;j<=k;j++)
            {
                ans=2e18;
                query(1,n);
                printf("%lld\n",ans); use[point.id]=1;
            }
        }
    }
    return 0;
}
```

# 总结

总的来说，K-D Tree可以应用于多维数点中，在偏序类题目中也有很大的作用，虽然复杂度比较玄学，但仍是拿高分的好方法，比如去年的APIO，用K-D Tree固然复杂度不对，但是也能拿到80+的高分，也许就是金银牌线的差别呢~~还是学好它吧，也是掌握了一种思想呢
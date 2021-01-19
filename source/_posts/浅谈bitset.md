---
title: 浅谈bitset
date: 2019-01-30 08:58:19
tags: [bitset,莫队]
categories: [学习笔记]
---

# 什么是bitset?

bitset是一种bug般的STL，可以用于骗分，卡常等，它实际上是一个类似布尔数组一样的东西，但是它每个位置只占1bit，而且可以整体移动（类似于状压，但是空间要小很多），而且，它能把你的时间复杂度/32或/64（看计算机的位数）。

<!--more-->

# 定义

首先，我们要知道怎样定义它，它存在于bitset库中，如果要定义的话，直接在<>内定义大小即可：

```c++
#include<bitset>
using namespace std;
int main()
{
    bitset<1000> s;//定义一个1000位的bitset
    return 0;
}
```

# 基本运算

实际上，bitset支持所有的位运算，基本实现跟状压差不多，但是，这是有时间消耗的：

例如，我们定义一个bitset< N > s，那么s<<k会消耗的时间复杂度为**O(N/w)**(w为计算机位数)。

# 常用函数

bitset中有很多内置的常用函数

```c++
bitset<N> s;
s.size()	返回大小（位数）
s.count()	返回1的个数（复杂度为O(N/w)）
s.any()		返回是否有1
s.none()	返回是否没有1
s.set()		全部变成1
s.set(p)	将p + 1位变成1（记住，是p+1为，因为bitset从0位开始）
s.set(p,x)	将p + 1为变成x
s.reset()	全部变成0
s.reset(p)	将p + 1位变成0
s.flip()	全部取反
s.flip(p)	将第p + 1为取反
s.to_ulong()	返回它转换为 unsigned long 的结果，超出范围则报错
s.to_ullong()	同上，返回 unsigned long long
s.to_string()	同上，返回 string
```

# 例题

[**小清新人渣的本愿**](https://www.luogu.org/problemnew/show/P3674)

题面就无力吐槽了，但是的确是一道bitset+莫队的好题

首先，对于差：我们考虑，我们有这样一个bitset，它记录了[l,r]中所有数有或无的状态，那么
$$
如果存在a,b使a-b=x,则a=b+x
$$
也就是说,b，b+x得同时存在，那么，我们用bitset，我们只需要考虑s&(s>>x)是否存在1就可以了；

对于和：我们考虑
$$
如果存在a,b使a+b=x,我们设b'=n-b,则a-b'=a+b-n=x-n
$$

$$
则b'=a+n-x
$$

这就是说，我们考虑再计一个bitset，s2，与原来的记录相反，是记录n-x的状况，那么，说明s1中的a与s2中的a+n-x同时存在，即我们只要判断s1&(s2>>(n-x))是否存在1就可以了；

而更令我们高兴的是，bitset中正好有一个s.any()，专门判断是否存在1，那么，就大功告成了；

最后，对于积：这个比较好办，我们只要枚举x的因数i（不超过$$\sqrt{x}$$个），再判断i与x/i是否同时存在就可以了。

其中，我们用莫队来不断更新s1,s2的值，就可以在$$O(\frac{nm}{w})$$的时间复杂度内算出答案（其实时间还是很紧的，但能过就行啦）

```c++
// luogu-judger-enable-o2
#include<cstdio>
#include<cmath>
#include<iostream>
#include<bitset>
#include<algorithm>
using namespace std;
const int maxN=1e5 + 100;
struct Node
{
    int op,l,r,x,id;
}q[maxN+1];
bitset<maxN> s1,s2;
int n,m,a[maxN+1];
int block,belong[maxN+1];
int cnt[maxN+1];
bool res[maxN+1];
inline int read()
{
    int num=0,f=1;
    char ch=getchar();
    while(!isdigit(ch)) {if(ch=='-') f=-1; ch=getchar();}
    while(isdigit(ch)) num=(num<<3)+(num<<1)+(ch^48),ch=getchar();
    return num*f;
}
bool comp(Node a,Node b)
{
    return belong[a.l]^belong[b.l] ? belong[a.l]<belong[b.l] : belong[a.l]&1 ? a.r<b.r : a.r>b.r;
}
inline void add(int x) {if(!cnt[x]) s1[x]=s2[maxN-x]=1; cnt[x]++;}
inline void del(int x) {cnt[x]--; if(!cnt[x]) s1[x]=s2[maxN-x]=0;}
void work()
{
    int l=1,r=0;
    for(int i=1;i<=m;i++)
    {
        while(l>q[i].l) add(a[--l]);
        while(r<q[i].r) add(a[++r]);
        while(l<q[i].l) del(a[l++]);
        while(r>q[i].r) del(a[r--]);
        int x=q[i].x;
        if(q[i].op==1) res[q[i].id]=(s1&(s1>>x)).any();
        else if(q[i].op==2) res[q[i].id]=(s1&(s2>>(maxN-x))).any();
            else
            {
                for(int j=1;j*j<=x;j++)
                    if(x%j==0)
                        if(s1[j]&&s1[x/j]) {res[q[i].id]=true; break;}
            }
    }
    for(int i=1;i<=m;i++) res[i]?puts("hana"):puts("bi");
}
int main()
{
    n=read(),m=read();
    block=n/sqrt(m*2/3);
    for(int i=1;i<=n;i++) a[i]=read(),belong[i]=(i-1)/block+1;
    for(int i=1;i<=m;i++) q[i].op=read(),q[i].l=read(),q[i].r=read(),q[i].x=read(),q[i].id=i;
    sort(q+1,q+m+1,comp);
    work();
    return 0;
}
```


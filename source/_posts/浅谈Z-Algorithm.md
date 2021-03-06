---
title: 浅谈Z_Algorithm
date: 2020-02-25 09:38:59
tags: [字符串]
categories: [学习笔记]
---

# Z_Algorithm（ex_kmp)

什么是$$Z$$算法？

对于一个字符串$$s$$，我们构造出一个数组$$z$$，$$z[i]$$表示$$s$$以$$i$$为开头的后缀与$$s$$的最长公共前缀（$$lcp$$）。

构造这个$$z$$数组的过程，就称为$$Z-Algorithm$$。

另外，$$Z-Algorithm$$是国外的说法，国内把称为$$exkmp$$，两者实际完全等价。

<!--more-->

# 构造方法

首先，可以确定的是$$z[1]=n$$，因为这个情况比较特殊，所以我们后面不做考虑。

然后，我们假设我们处理到第$$i$$位，定义$$r$$表示之前所有处理的位数（不包括$$1$$）中$$j+z[j]-1$$最大的值，记录$$l$$为当$$z[j]$$为$$r$$时的$$j$$的值。

我们的目标，是像$$kmp$$一样，尽量地用上我们已经算出来的值。

首先，如果$$i>r$$，那么没有办法，只能从当前位置一个个暴力匹配，最后看$$l,r$$是否要更新即可。

如果$$i<=r$$，因为$$s[l,r]$$存在$$s$$的一个前缀与它相等，那么说明$$s[i,r]$$在之前出现过！那么，我们就可以利用之前的答案！考虑$$s[i,r]$$之前出现在哪，容易发现，就是$$s[i-l+1,r-l+1]$$（因为$$s[l,i-1]$$对应的是$$s[1,i-l]$$），那么我们就可以利用$$z[i-l+1]$$的信息！

那么，我们就可以令$$z[i]=min(z[i-l+ 1],r-i+1)$$，为什么要取$$min$$？因为我们只能保证$$s[i,r]=s[i-l+1,r-l+1]$$，但是再后面就无从得知了，所以，后面还要暴力匹配看$$z[i]$$能否扩大。

最后，我们再用现有的$$i,z[i]$$看能否更新$$l,r$$即可！

# Code

```c++
inline void get_z()
{
    z[1] = n;
    for(int i = 2, l = 0, r = 0; i <= n; i++)
    {
    	if(i <= r) z[i] = min(z[i - l + 1], r - i + 1);
        while(i + z[i] <= n && s[ i + z[i] ] == s[ z[i] + 1 ]) z[i] ++;
        if(i + z[i] - 1 > r) l = i, r = i + z[i] - 1;
    }
}
```

# 时间复杂度

时间复杂度，是$$O(n)$$的。

为什么呢？

证明：

我们只需要判断内层$$while$$循环的复杂度，因为别的都是常数。

我们来证明，每次只要运行内层$$while$$循环，必然会使$$r$$增加。

我们分情况讨论：

首先，如果$$i>r$$，那么显然，每次循环必然会使$$r$$增加，无需讨论。

如果$$i<=r$$，那么我们设一开始的$$z[i]$$值为$$z_0$$，那么有两种情况：

一、 $$z_0=r-i+1$$，那么我们每次循环，使得$$z[i]$$增加，必然也会使得$$r$$增加（因为$$r=i+z[i]-1$$）。

二、如果$$z_0 < r-i+1$$，那么$$z_0=z[i-l+1]$$，那么，我们来证明此时必然不会循环。

考虑反证法，如果有循环使得$$z[i]>z_0$$，那么就说明存在$$z_0+1$$使得$$s[z_0+1]=s[i+z_0]$$，而因为$$i+z_0<r+1$$（假设条件），可以得知$$i+z_0<=r$$，而这个位置还在$$[i,r]$$以内，也就是说以$$i-l+1$$为开头的后缀中也有这个字符，那么$$z[i-l+1]$$就不可能是$$z_0$$，矛盾！

所以此时循环不会进行。

综上，我们证明了每次循环都必定会让$$r$$增加，而$$r$$最多只会增长$$n$$次，所以时间复杂度是$$O(n)$$的。

# 应用

我们只求出了对于一个字符串$$s$$的$$z$$是没有用的，我们要通过这个来求另一个字符串$$t$$的所有后缀与$$s$$的最长公共前缀。

方法很简单，就是我们对$$s+a+t$$（$$a$$是某个$$s$$和$$t$$都没有的字符）整个跑一遍$$Z-Algorithm$$，然后对$$t$$中每个位置直接访问其在新串中的位置的$$z$$数组就可以了。


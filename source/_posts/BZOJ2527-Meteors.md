---
title: '[BZOJ2527]Meteors'
date: 2019-09-13 17:13:51
tags: [整体二分]
categories: [BZOJ]
---

# Description

某地有$$N$$个成员国。现在它发现了一颗新的星球，这颗星球的轨道被分为M份（第M份和第1份相邻），第i份上有第Ai个国家的太空站。
这个星球经常会下陨石雨。BIU已经预测了接下来K场陨石雨的情况。
BIU的第i个成员国希望能够收集Pi单位的陨石样本。你的任务是判断对于每个国家，它需要在第几次陨石雨之后，才能收集足够的陨石。

<!--more-->

# Solution

整体二分。

复杂度$$O(nlog^2n)$$

注意：中间加的时候可能会爆**long long**。

# Code

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxN = 3e5 + 100;

struct Node
{
    int id, num;
}q[maxN + 1], qr[maxN * 2 + 1];

int C[maxN * 4 + 1];
int n, m, k;
int ans[maxN + 1];
int L[maxN + 1], R[maxN + 1], val[maxN + 1];

vector<int> place[maxN + 1];

inline int read()
{
    int num = 0, f = 1;
    char ch = getchar();
    while( !isdigit( ch ) ) { if(ch == '-') f = -1; ch = getchar(); }
    while( isdigit( ch ) ) num = (num << 3ll) + (num << 1ll) + (ch ^ 48), ch = getchar();
    return num * f;
}

inline int lowbit(int x) { return x & (-x); }

inline void change(int x, int num)
{
    for(int i = x; i <= m; i += lowbit(i)) C[i] += num;
}

inline int query(int x)
{
    int ans = 0;
    for(int i = x; i > 0; i -= lowbit(i)) ans += C[i];
    return ans;
}

inline void solve(int l, int r, int x, int y)
{
    if(l > r || x > y) return;
    if(l == r)
    {
        for(int i = x; i <= y; i++) ans[ q[i].id ] = l;
        return;
    }
    int mid = (l + r) >> 1, t1 = 0, t2 = n;
    for(int i = l; i <= mid; i++) 
    {
        if(L[i] > R[i]) change(1, val[i]);
        change(L[i], val[i]), change(R[i] + 1, -val[i]);
    }
    for(int i = x; i <= y; i++)
    {
        int t = q[i].id, res = 0;
        for(int j = 0; j < place[t].size(); j++)
        {
            res += query(place[t][j]);
            if(res >= q[i].num) break;//就是这里
        }
        if(res >= q[i].num) qr[++ t1] = q[i];
        else qr[++ t2] = q[i], qr[t2].num -= res;
    }
    for(int i = l; i <= mid; i++) 
    {
        if(L[i] > R[i]) change(1, -val[i]);
        change(L[i], -val[i]), change(R[i] + 1, val[i]);
    }
    for(int i = x; i <= x + t1 - 1; i++) q[i] = qr[i - x + 1];
    for(int i = x + t1; i <= y; i++) q[i] = qr[n + i - x - t1 + 1];
    solve(l, mid, x, x + t1 - 1); solve(mid + 1, r, x + t1, y); 
}

#undef int
int main()
#define int long long
{
    n = read(), m = read();
    for(int i = 1; i <= m; i++) place[ read() ].push_back(i);
    for(int i = 1; i <= n; i++) q[i].id = i, q[i].num = read();
    k = read();
    for(int i = 1; i <= k; i++) L[i] = read(), R[i] = read(), val[i] = read();
    solve(1, k + 1, 1, n);
    for(int i = 1; i <= n; i++)
       if(ans[i] == k + 1) puts("NIE");
       else printf("%lld\n", ans[i]);
    return 0;
}
```


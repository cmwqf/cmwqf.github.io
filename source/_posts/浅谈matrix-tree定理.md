---
title: 浅谈Matrix-Tree定理
date: 2019-02-03 11:00:27
tags: [线性代数]
categories: [学习笔记]
---

# 行列式

## 定义

行列式指一个n阶方阵（行数和列数相等）的值，记为$$det(A)$$或$$|A|$$。

求行列式的公式为:
$$
det(K)=\sum_P((-1)^{T(P)}*K_{1,p1}...*K_{n,pn})
$$
其中，P为1~N任意的一个排列，T(P)表示排列P的逆序对数，那个求和式的每一项就是从矩阵中选出n个数，使他们的行列都不重合。

<!--more-->

## 性质

1. 交换矩阵的两行（列），行列式变号

2. 如果矩阵有两行（列）完全相同，则行列式为0

3. 如果某一行（列）都乘以同一个数k，那么新行列式的值等于原行列式的值乘k

4. 如果矩阵有两行（列）成比例（比例系数为k），那么行列式为0

5. 如果把矩阵某一行（列）加上另一行（列）的k倍，行列式的值不变

## 求法

对于一个行列式，我们根据**性质5**，将它转化为一个上三角矩阵（左下半部分全是0），那么，**其行列式的值就等于对角线上数的乘积**（带入求行列式公式即证）。

|  2   |  2   |  3   |  1   |
| :--: | :--: | :--: | :--: |
|  0   |  3   |  1   |  2   |
|  0   |  0   |  1   |  3   |
|  0   |  0   |  0   |  4   |

这就是一个上三角矩阵。

这样，我们通过高斯消元的办法，把一个矩阵化成这个形式，再把对角线乘起来就可以了。

如果矩阵不可以出现实数，且要取模的话，就要用辗转相除高斯消元法。

# Matrix-Tree定理

这是一个计算一个图的生成树个数的定理，它的计算规则如下：

设D为度数矩阵，其中
$$
D[i][j]=0(i\ne j),D[i][i]=i号点的度数
$$
还有一个邻接矩阵A（这个大家都知道是啥吧）
$$
A[i][i]=0,A[i][j]=A[j][i]=i,j的边数
$$
定义**基尔霍夫矩阵**K=D-A

那么，最后的答案就是K**去掉第i行和第i列后的行列式（称为余子式）的值**。

# 例题

[小Z的房间](https://www.luogu.org/problemnew/show/P4111)

就是一个裸的Matrix-Tree定理的模板。

```c++
#include<cstdio>
#include<iostream> 
#define int long long
using namespace std;
const int maxN=5000,mod=1e9,step[2][2]={{-1,0},{0,-1}};
int tot,n,m;
int a[maxN+1][maxN+1],id[maxN+1][maxN+1];
char G[maxN+1][maxN+1];
inline int det()
{
    int ans=1;
    for(int i=1;i<=tot;i++)
    {
        for(int j=i+1;j<=tot;j++)//把i+1->tot行的第i列全部消成0 
            while(a[j][i])
            {
                int tmp=a[i][i]/a[j][i];
                for(int k=i;k<=tot;k++) 
                {
                    a[i][k]=((a[i][k]-tmp*a[j][k])%mod+mod)%mod;
                    swap(a[i][k],a[j][k]);
                }
                ans*=-1;
            }
        if(a[i][i]==0) return 0;
        ans=(ans*a[i][i]%mod+mod)%mod;
    }
    return (ans%mod+mod)%mod;
}
#undef int
int main()
#define int long long
{
    scanf("%lld%lld",&n,&m);
    for(int i=1;i<=n;i++) scanf("%s",G[i]+1);
    for(int i=1;i<=n;i++)
        for(int j=1;j<=m;j++)
            if(G[i][j]=='.') id[i][j]=++tot;
    for(int i=1;i<=n;i++)
        for(int j=1;j<=m;j++)
            if(G[i][j]=='.')
                for(int k=0;k<2;k++)
                {
                    int x=i+step[k][0],y=j+step[k][1];
                    if(x<1||y<1||x>n||y>m||G[x][y]=='*') continue;
                    int u=id[i][j],v=id[x][y];
                    a[u][u]++; a[v][v]++;
                    a[u][v]--; a[v][u]--;
                }
    tot--;
    printf("%lld",det());
    return 0;
} 
```


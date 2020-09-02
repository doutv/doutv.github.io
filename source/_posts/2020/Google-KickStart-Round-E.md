---
title: Google-KickStart-Round-E
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
categories:
  - - Algorithm
date: 2020-08-27 08:01:54
summary:
---

# Google KickStart Round E
这次做的有点菜😓

## High Buildings
题意：构造一个以`a[1]`为首且长度为`a`的最长不下降序列，以`a[n]`为尾且长度为`b`的最长不上升序列，这两个序列的公共部分长度为`c`。

考场上不知道哪里的边界情况没处理好，而且没利用好不下降/不上升的一个性质：$a[i]\leq a[i+1]$，可以构造为$a[i]=a[i+1]$。以下代码参考自榜1。
```cpp
#include <iostream>
#include <cmath>
#include <algorithm>
#include <vector>
#include <map>
#include <stack>
#include <queue>
#include <unordered_map>
#include <string>
using namespace std;

typedef long long ll;

void solve()
{
    int n, a, b, c;
    cin >> n >> a >> b >> c;
    int lena = a - c;
    int lenb = b - c;
    if (lena + lenb + c > n || (lena + lenb + c == 1 && n >= 2))
    {
        printf("IMPOSSIBLE\n");
        return;
    }
    if (n == 1)
    {
        printf("1\n");
        return;
    }
    if (n == 2)
    {
        if (c == 2)
            printf("1 1\n");
        else if (a == 2)
            printf("1 2\n");
        else if (b == 2)
            printf("2 1\n");
        return;
    }
    vector<int> ans;
    for (int i = 1; i <= lena; i++)
        ans.push_back(2);
    for (int i = 1; i <= c; i++)
        ans.push_back(3);
    for (int i = 1; i <= lenb; i++)
        ans.push_back(2);
    int extra = n - lena - lenb - c;
    if (extra)
        ans.insert(ans.begin() + 1, extra, 1);
    for (int i = 0; i < n; i++)
        printf("%d ", ans[i]);
    printf("\n");
}

int main()
{
    int T;
    cin >> T;
    cin.ignore();
    for (int caseNum = 1; caseNum <= T; caseNum++)
    {
        printf("Case #%d: ", caseNum);
        solve();
    }
    return 0;
}
```

## Toys
这题想了一会正解，没想出来，又懒得写暴力，所以就没做了。看了正解之后发现，其实我已经想到一大半了......

如果一个toy不能无限玩：$R_i>\sum_{j\neq i}E_j=\sum_j E_j-E_i$，所以能无限玩的toy满足：$R_i+E_i\leq SUM$, where $SUM$ is the sum of all selected toys $E_i$.

最优解：每次尝试移除$R_i+E_i$最大的toy，因为如果移除其他的toy，such as $j$, $R_i+E_i>SUM$依然成立。


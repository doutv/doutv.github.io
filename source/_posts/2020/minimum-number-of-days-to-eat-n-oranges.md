---
title: minimum-number-of-days-to-eat-n-oranges
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
categories:
  - - Algorithm
date: 2020-08-16 17:56:14
summary:
---

# [5490. 吃掉 N 个橘子的最少天数](https://leetcode-cn.com/problems/minimum-number-of-days-to-eat-n-oranges/)
一道平平无奇的记忆化搜索题目，题面已经把递归方程给我们了。

现在的问题是$n\leq 2e9$，如何实现记忆化呢？这里可以用到$unordered\_map$来存储状态，因为所需要的状态远远小于$2e9$，直接开一个$2e9$大小的数组是不可能的。

## code
```cpp
class Solution {
public:
    unordered_map<int,int> dp;
    int f(int n)
    {
        if (n==1)
            return 1;
        if (n==2||n==3)
            return 2;
        if (dp.find(n)!=dp.end())
            return dp[n];
        int res=min(f(n/2)+n%2+1,f(n/3)+n%3+1);
        dp[n]=res;
        return res;
    }
    int minDays(int n) {
        dp.clear();
        return f(n);
    }
};
```

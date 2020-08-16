---
title: number-of-sub-arrays-with-odd-sum
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
categories:
  - - Algorithm
date: 2020-07-27 18:17:13
summary:
---

# [1524.和为奇数的子数组数目](https://leetcode-cn.com/problems/number-of-sub-arrays-with-odd-sum/)
题目看起来很简单，但是做起来可就不简单了

我的思路是：奇数+偶数=奇数，奇数=奇数，根据这个规律来找dp方程，想了两个dp方程都不对......

一看题解，阔然开朗。
1. 利用前缀和的性质：$sum[i..j]=sum[j]-sum[i-1]$。
2. 奇数-偶数=奇数，偶数-奇数=奇数。
3. 所以我们只需要记录$[0...i-1]$的前缀和中有多少个奇数$odd$和偶数$even$。如果$sum[i]$是奇数，那么以$arr[i]$为结尾的子数组和为奇数的数目为$even$；如果$sum[i]$是偶数，则为$odd$。

[代码来源于评论区](https://leetcode-cn.com/problems/number-of-sub-arrays-with-odd-sum/comments/)
```cpp
class Solution {
public:
    int numOfSubarrays(vector<int>& arr) {
        const int mod=1e9+7;
        int len = arr.size();
        int even = 1, odd = 0;  // 前缀和奇偶个数
        long res = 0, sum = 0;
        // 利用前缀和，将之前的前缀和的奇偶数量记录下来，根据当前子数组的奇偶，
        //再使用之前的更小子数组奇偶个数算出以当前数为尾的子数组的奇和个数
        for (int i = 0; i < len; i++) {
            sum += arr[i];
            if (sum % 2 == 0) {
                res += odd;
                res %= mod;
                even++;
            } else {
                res += even;
                res %= mod;
                odd++;
            }
        }
        return res;
    }
};
```
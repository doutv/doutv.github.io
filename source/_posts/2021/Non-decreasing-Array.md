---
title: Non-decreasing-Array
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
  - LeetCode
  - Greedy
categories:
  - - Algorithm
date: 2021-02-08 16:07:03
summary:
---

## 解题思路
解题关键是遇到$nums[i]>nums[i+1]$时，应该怎么处理？

1. 找到第一处 $nums[i]>nums[i+1]$ ，改 $nums[i]$ 还是 $nums[i+1]$ 呢？
   1. 改 $nums[i]=nums[i+1]$ : 当且仅当 $nums[i-1]\leq nums[i+1]$ 时，因为需要保证 $nums[i-1] \leq nums[i] = nums[i+1]$
   2. 改 $nums[i+1] = nums[i]$ , $nums[i-1] \leq nums[i] = nums[i+1]$ 一定成立
   3. 然而我们应该优先改第一种情况 $nums[i]=nums[i+1]$ ，因为这样做是无后效性的，而第二种情况更改了 $nums[i+1]$ 的值，可能会破坏后面的单调递增序列，如这样一个序列： `1 100 2 3`


## 代码

```cpp
class Solution {
public:
    bool checkPossibility(vector<int>& nums) {
        int cnt=0;
        for (int i=0;i<nums.size()-1;i++)
        {
            if (nums[i]>nums[i+1])
            {
                if (cnt)
                    return 0;
                ++cnt;
                if (i==0)
                    continue;
                if (i>0 && nums[i-1]<=nums[i+1])
                {
                    nums[i]=nums[i+1];
                    continue;
                }
                nums[i+1]=nums[i];
            }
        }
        return 1;
    }
};
```


## 总结
分析完才发现这居然是一道贪心题。  
做这种题不能面向用例测试，若是在面试时遇到这题我估计就凉了，简单题一点也不简单呀，不能小看简单题！我们需要认真拿出纸笔分析题目。
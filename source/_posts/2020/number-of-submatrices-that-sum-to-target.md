---
title: number-of-submatrices-that-sum-to-target
date: 2020-07-05 13:14:48
tags: 
    - Algorithm
    - Leetcode
    - sort
    - greedy
mathjax: true
categories:
    - [Algorithm,greedy]
---

[5455. 最多 K 次交换相邻数位后得到的最小整数](https://leetcode-cn.com/problems/number-of-submatrices-that-sum-to-target/)

给你一个字符串 num 和一个整数 k 。其中，num 表示一个很大的整数，字符串中的每个字符依次对应整数上的各个数位 。
你可以交换这个整数相邻数位的数字 最多 k 次。
请你返回你能得到的最小整数，并以字符串形式返回。

<!-- more -->

1. 我第一反应是冒泡排序，因为只能交换相邻元素。
2. 敲完冒泡排序发现样例1过不去，手推了一会，发现是因为冒泡排序可能会在某一轮中断。
3. 然后我想到了先贪心后冒泡排序，冒泡排序的交换次数$\sum_{i=n-1}^x i<k$（后来发现这个交换次数是错误的，这只是交换次数的上界），先找到满足不等式最大的$x$，贪心交换$k-\sum_{i=n-1}^x i$ 轮后再进行冒泡排序。
4. 上面那个方案当然过不了样例5啦，比赛快结束的时候，想到了要在每一轮冒泡排序前，计算出该轮所需的交换次数，离AC很近了。
5. 比赛结束前还是没做出来，后来看到[其实就是活用一点冒泡排序](https://leetcode-cn.com/problems/minimum-possible-integer-after-at-most-k-adjacent-swaps-on-digits/solution/5455qi-shi-jiu-shi-huo-yong-yi-dian-mou-pao-pai-xu/)这个题解，阔然开朗，只要每一轮的冒泡排序只考虑$k$满足的区间$[i,i+k]$即可，
6. 代码：
```cpp
class Solution
{
public:
    string minInteger(string num, int k)
    {
        int n = num.size();
        if (k > n * (n - 1) / 2)
        {
            sort(num.begin(), num.end());
            return num;
        }
        for (int i = 0; i < n; i++)
        {
            int minx = num[i];
            int idx = i;
            for (int j = i + 1; j < n && j - i <= k; j++)
                if (num[j] < minx)
                {
                    minx = num[j];
                    idx = j;
                }
            for (int j = idx; j >= i + 1; j--)
            {
                swap(num[j], num[j - 1]);
                --k;
            }
        }
        return num;
    }
};
```

以上解法是$O(n^2)$的，事实上这题有$O(n\log n)$的解法：贪心地填每一位数，用线段树/树状数组维护置换次数，坑待填~，可参考：
+ [线段树](https://leetcode-cn.com/problems/minimum-possible-integer-after-at-most-k-adjacent-swaps-on-digits/solution/java-rmqfenwich-tree-onlgn-by-henrylee4/)
+ [树状数组](https://www.bilibili.com/video/BV1tD4y1Q7Mb?p=5)
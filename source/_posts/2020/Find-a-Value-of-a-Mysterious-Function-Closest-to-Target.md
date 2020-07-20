---
title: Find a Value of a Mysterious Function Closest to Target
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
  - Leetcode
  - Competition
categories:
  - - Algorithm
date: 2020-07-20 14:06:18
summary:
---

# [1521. 找到最接近目标值的函数值](https://leetcode-cn.com/problems/find-a-value-of-a-mysterious-function-closest-to-target/)
当时卡在第三题了，第四题还没看过😓

后面看了一下题目，感觉跟前缀和相关，然后懒懒的我就去看题解了~

## 位运算
[这个题解](https://leetcode-cn.com/problems/find-a-value-of-a-mysterious-function-closest-to-target/solution/10-xing-dai-ma-an-wei-yu-yun-suan-de-xing-zhi-by-2/)讲得很好，建议认真吸收。

根据按位与的两个性质，尤其是第二个性质，使得我们可以轻松地遍历每一个`func(arr,l,r)`的值，从而得到答案。

代码直接看上面题解就好，我基本照抄的。
```cpp
class Solution {
public:
    int closestToTarget(vector<int>& arr, int target) {
        int ans=abs(arr[0]-target);
        vector<int> valid{arr[0]};
        for (int i=1;i<arr.size();i++)
        {
            vector<int> new_valid;
            for (int v:valid)
                new_valid.emplace_back(v&arr[i]);
            new_valid.emplace_back(arr[i]);       
            new_valid.erase(unique(new_valid.begin(),new_valid.end()),new_valid.end());
            for (int v:new_valid)
                ans=min(ans,abs(v-target));
        }
        return ans;
    }
};
```

## st表+二分
这里同样需要用到按位与&的一个性质：$x\&y \geq x$ and $x\&y \geq y$。所以有单调性$func(arr,l,r)\geq func(arr,l,r+1)$。

题目要找的最小值$ans$就是最接近$target$的$func(arr,l,r)$，显然这个最小值$ans$一定出现在比$target$小的第一个$func$或比$target$大的第一个$func$。因此我们可以二分查找出比$target$小的第一个$func(arr,l,r)$，再选出$|func(arr,l,r-1)-target|$和$|func(arr,l,r)-target|$两者中的最小值。

复习一下关于st表的知识：  
- https://oi-wiki.org/ds/sparse-table/
- st表是用于解决可重复贡献问题的数据结构。
- 可重复贡献问题指对于运算$opt$，满足$x opt x = x$，$opt$还需要满足结合律才可用st表求解。
- 能用st表解决的可重复贡献问题有：最大值$\max(x,x)=x$，GCD，按位与。
- st表可以让我们在$O(1)$内求出某个区间$[L,R]$内的最大$func(arr,l,r)$ $L\leq l \leq r \leq R$。

再根据$func$的单调性，固定左端点$l$，二分查找$r$使$|func(arr,l,r)-target|$最小。枚举$l$需要$O(N)$，二分查找需要$O(\log N)$，st表查找一次需要$O(1)$,所以总的复杂度是$O(N)*O(\log N)*O(1)=O(N\log N)$。

[参考题解](https://leetcode-cn.com/problems/find-a-value-of-a-mysterious-function-closest-to-target/solution/mei-ju-qi-dian-rmq-er-fen-by-acw_wangdh15/)
```cpp
class Solution {
public:
    int closestToTarget(vector<int>& arr, int target) 
    {
        int Logn[100005];
        int ans=INT32_MAX;
        int n=arr.size();
        vector<vector<int>> st(n,vector<int>(20));
        for (int i=0;i<n;i++)
            st[i][0]=arr[i];
        Logn[1] = 0;
        Logn[2] = 1;
        for (int i = 3; i <= n; i++) 
            Logn[i] = Logn[i / 2] + 1;
        for (int j=1;j<20;j++)
            for (int i=0;i+(1<<j)-1<n;i++)
                st[i][j]=st[i][j-1] & st[i+(1<<(j-1))][j-1];
        // 现学的lambda function 
        auto ask = [st,Logn](int x,int y)
        {
            if (x>y)
                return INT32_MAX;
            int tmp=Logn[y-x+1];
            return st[x][tmp] & st[y-(1<<tmp)+1][tmp];
        };
        for (int l=0;l<n;l++)
        {
            int r_left=l;
            int r_right=n-1;
            while (r_left<r_right)
            {
                int mid=(r_left+r_right)/2;
                if (ask(l,mid)>=target)
                    r_left=mid+1;
                else
                    r_right=mid;
            }
            int r=r_right;
            ans=min(ans,abs(target-ask(l,r)));
            ans=min(ans,abs(target-ask(l,r-1)));
        }
        return ans;
    }
};
```


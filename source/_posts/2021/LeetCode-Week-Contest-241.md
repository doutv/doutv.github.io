---
title: LeetCode Week Contest 241
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
categories:
  - - Algorithm
date: 2021-05-16 15:49:50
summary:
---

# LeetCode 241 周赛
> 好久没写题解&总结了T^T

## [5759. 找出所有子集的异或总和再求和](https://leetcode-cn.com/problems/sum-of-all-subset-xor-totals/)

### dfs遍历
开始写的时候还在想怎么遍历数组的所有子集，后来憋了个dfs出来：
```cpp
class Solution {
public:
    int calSubset(vector<int>& nums,int cur,int xorval)
    {
        if (cur >= nums.size())
            return 0;
        int sum = 0;
        xorval ^= nums[cur];
        sum += xorval;
        // 尝试选择后面的任意一个数字进行组合
        for (int i=cur+1;i<nums.size();i++)
            sum += calSubset(nums,i,xorval);
        return sum;
    }
    int subsetXORSum(vector<int>& nums) {
        int res = 0;
        // 子集可以从 [0,n) 任意数字开始
        for (int i=0;i<nums.size();i++)
            res += calSubset(nums,i,0);
        return res;
    }
};
```

后来看题解看到了两种很好的办法：
### [二进制枚举所有子集](https://leetcode-cn.com/problems/sum-of-all-subset-xor-totals/solution/c-er-jin-zhi-mei-ju-zi-ji-mo-ban-ti-by-e-ab05/)

- 用一个长度等于数组长度的二进制数来表示子集的情况，这个二进制数的每一位对应数组每一位是取还是不取（取为1，不取为0）。  
- e.g. `1001` 表示取数组的第一位和第四位
- 这个二进制数的所有可能的取值与数组的所有子集是一一对应的，所以我们只需要从`0`枚举到`1<<数组长度n`

```cpp
class Solution {
public:
    int subsetXORSum(vector<int>& nums) {
        int n = nums.size();
        int ans = 0;
        // 枚举所有子集
        for (int i=0;i<(1<<n);i++)
        {
            // 枚举子集中要取的位置
            int cur = 0;
            for (int j=0;j<n;j++)
            {
                if (i & (1<<j))
                    cur ^= nums[j];
            }
            ans += cur;
        }
        return ans;
    }
};
```

### 数学方法

- 我们考虑最终结果中每个bit的来源
- 证明见：https://leetcode-cn.com/circle/discuss/COaQbh/view/z7mqn9/
- （反正我还没想懂）
- 代码来源：https://leetcode-cn.com/problems/sum-of-all-subset-xor-totals/solution/

```cpp
class Solution {
public:
	int subsetXORSum(vector<int>& a) {
		int t=0; 
        for (auto i:a)
            t|=i;
		return t<<(a.size()-1);
	}
};
```

## [5760. 构成交替字符串需要的最小交换次数](https://leetcode-cn.com/problems/minimum-number-of-swaps-to-make-the-binary-string-alternating/)

题意：将任意一个01字符串串转化为01必须相邻的字符串，可任意交换两个字符

1. 先判断是否有解，有解的条件：
   1. 字符串长度为奇数：0和1的个数相差1
   2. 字符串长度为偶数：0和1的个数相等
2. 求最小交换次数
   1. 我们不需要关心具体交换了哪两个字符，因为所有01都是等价的，可任意交换的。
   2. 目标字符串是已知的
      1. 字符串长度为奇数："010...010" 或 "101...101"
      2. 字符串长度为偶数："101...010" 或 "010...010"
      3. 所以无论字符串长度是奇数还是偶数，目标字符串只有两种：
         1. "0101..."
         2. "1010..."
   3. 我们只需要比较当前字符串与目标字符串不同的字符数量即可
      1. 因为每交换一次会使不同的字符数量 `-2`
      2. 如果不同的字符数量是奇数的话，那说明我们是无法得到这个目标字符串的
      3. 如果不同的字符数量是偶数的话，交换次数就是不同的字符数量/2
```cpp
class Solution {
public:
    int minSwaps(string s) {
        int cnt1 = 0;
        int cnt0 = 0;
        int ans = 0;
        // 判断是否有解
        for (char ch:s)
        {
            if (ch == '0')
                ++cnt0;
            else
                ++cnt1;
        }
        if ((s.size() & 1) && !(abs(cnt0-cnt1)==1))
            return -1;
        if (!(s.size() & 1) && (cnt0 != cnt1))
            return -1;

        // 求最小交换次数
        // 求当前字符串与 "1010..." 的不同的字符数量
        int ans0 = 0;
        for (int i=0;i<s.size();i++)
        {
            char ch = (i&1)? '0':'1';
            if (s[i] != ch)
                ++ans0;
        }
        // 求当前字符串与 "0101..." 的不同的字符数量
        int ans1=0;
        for (int i=0;i<s.size();i++)
        {
            char ch = (i&1)? '1':'0';
            if (s[i] != ch)
                ++ans1;
        }
        // 因为每交换一次会使不同的字符数量-2
        // 如果不同的字符数量是奇数的话，那说明我们是无法得到这个目标字符串的
        // 如果不同的字符数量是偶数的话，交换次数就是不同的字符数量/2
        ans0 = (ans0 & 1)? s.size():(ans0>>1);
        ans1 = (ans1 & 1)? s.size():(ans1>>1);
        ans = min(ans1,ans0);
        return ans;
    }
};
```

## [5761. 找出和为指定值的下标对](https://leetcode-cn.com/problems/finding-pairs-with-a-certain-sum/)
- 注意观察数据范围，用哈希表即可解决。
- 先给nums1排序，这样在`count()`中当`tot-x < 1`时可以直接结束
```cpp
class FindSumPairs {
public:
    vector<int> a,b;
    unordered_map<int,int> m;
    FindSumPairs(vector<int>& nums1, vector<int>& nums2) : a(nums1),b(nums2){
        sort(a.begin(),a.end());
        for (int each:b)
            ++m[each];
    }
    
    void add(int index, int val) {
        --m[b[index]];
        b[index] += val;
        ++m[b[index]];
    }
    
    int count(int tot) {
        int res = 0;
        for (int x:a)
        {
            if (tot-x < 1)
                break;
            res += m[tot-x];
        }
        return res;
    }
};
```

## [5762. 恰有 K 根木棍可以看到的排列数目](https://leetcode-cn.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/)
- 当时时间不太够，猜是斯特林数，去查了查觉得也不太像，于是就去吃饭啦

### DP
- 状态定义：`f(i,j)`表示用长度从`1`到`i`的木棍排成一排，从左侧可以看到恰好`j`根木棍的方案数
- 最优子结构：
  - 如何证明？
- 状态转移
  - `f(i,j) = f(i-1,j-1) + (i-1)*f(i-1,j)`
  - `i-1` -> `i` `j`不变：在原`i-1`根木棍中插入，在原方案数上进行排列
  - `i-1` -> `i` `j-1` -> `j`：加入长度为`i`的木棍，让它排在最后面

```cpp
class Solution {
public:
    typedef long long ll;
    ll f[1005][1005];
    int rearrangeSticks(int n, int k) {
        f[0][0] = 1;
        const ll mod = 1000000007;
        for (int i=1;i<=n;i++)
        {
            for (int j=1;j<=i;j++)
            {
                f[i][j] = f[i-1][j-1] + (i-1)*f[i-1][j];
                f[i][j] %= mod;
            }
        }
        return f[n][k];
    }
};
```
### 斯特林数
- 见：https://leetcode-cn.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/solution/zhuan-huan-cheng-di-yi-lei-si-te-lin-shu-2y1k/
- https://zh.wikipedia.org/wiki/%E6%96%AF%E7%89%B9%E7%81%B5%E6%95%B0
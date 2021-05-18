---
title: Maximum Number of Non-Overlapping Substrings
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
  - LeetCode
  - Competition
  - greedy
categories:
  - - Algorithm
date: 2020-07-20 09:44:02
summary:
---

# [5466. Maximum Number of Non-Overlapping Substrings](https://leetcode-cn.com/problems/maximum-number-of-non-overlapping-substrings/)
Leetcode周赛P3，才一百多人做出来，题目很新颖，并不需要用到很高深的算法，但是对思维要求高。

1. 一开始我是想着贪心的，我的想法是子串只包含一种字符，这样构造得到的子串应该是最多的。算法大概是枚举每一个出现的字符的右端点`last[ch]`，向左扩展到左端点，要求这段字符串满足条件2。交上去之后发现`"abab"`的答案是`["abab"]`，我的贪心是过不了这个样例的。
2. 于是我就去想dp的做法了，`dp[i]`表示前$[0,i]$中的最多子串数目，$dp[i]=\max{dp[j]+1}$ 如果子串$[j+1,i]$所有出现的字符都只出现在子串中，对于所有的$i$, $dp[i]=dp[i-1]$. 可以用一个优先队列来保存之前的`dp[j]`，每次只取$\geq dp[i-1]$的出来，检查$[j+1,i]$区间是否符合条件。然而，这样做连第一个样例都过不去😓。首先，复杂度可能会达到$O(N^2)$，其次，要得到最优解的字符串比较难。把代码放这纪念吧...
```cpp
vector<string> maxNumOfSubstrings(string s)
{
    vector<string> ans;
    int n = s.size();
    vector<int> dp(n, 0);
    vector<int> fromdp(n, -1);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> q;
    int first[30], last[30];
    memset(first,-1,sizeof(first));
    memset(last,-1,sizeof(last));
    for (int i = 'a'; i <= 'z'; i++)
    {
        first[i - 'a'] = s.find(i);
        last[i - 'a'] = s.rfind(i);
    }
    for (int i = 0; i < n; i++)
    {
        q.push({0, i-1});
        dp[i]=i?dp[i-1]:0;
        queue<pair<int, int>> tmpq;
        while (!q.empty())
        {
            auto [pre, le] = q.top();
            q.pop();
            if (pre < dp[i])
                break;
            bool bo = 0;
            printf("%d %d %d\n",i,le,pre);
            for (int j = le + 1; j <= i; j++)
            {
                int ch = s[j] - 'a';
                if (first[ch] > le && last[ch] <= i)
                    continue;
                else
                {
                    bo = 1;
                    break;
                }
            }
            tmpq.push({pre, le});
            if (bo)
                continue;
            else
            {
                dp[i] = pre + 1;
                fromdp[i] = pre;
                q.push({dp[i], i});
                break;
            }
        }
        while (!tmpq.empty())
        {
            q.push(tmpq.front());
            tmpq.pop();
        }
    }
    for (int i=0;i<n;i++)
        cout<<dp[i]<<" ";
    for (int i = n - 1; i != -1; i = fromdp[i])
    {
        ans.push_back(s.substr(fromdp[i] + 1, i - fromdp[i]));
    }
    return ans;
}
```
3. 评论区的贪心解法$O(N)$:
   1. 预处理出`l[ch]`表示字符`ch`最左侧的出现位置, `r[ch]`表示最右侧的出现位置。
   2. `get`函数固定子串的左端点`start`，求至少包含左端点的字符`s[start]`的合法子串的右端点。
   3. 遍历字符串，只找以当前字符`s[i]`在字符串第一次出现的字符开始的字符串，用一个神奇的变量`right`来记录当前答案中最右侧字符串的右端点位置。如果`i>right`说明已经走出了上一个字符串的范围了，可以开始下一个字符串了。
   4. 贪心的正确性证明：我不知道怎么证明😂
```cpp
class Solution
{
public:
    vector<string> maxNumOfSubstrings(string s)
    {
        int l[26], r[26];
        memset(l, -1, sizeof l);
        memset(r, -1, sizeof r);
        for (int i = 0; i < s.size(); i++)
        { //记录每个字符第一次和最后一次出现的位置
            int x = s[i] - 'a';
            if (l[x] == -1)
                l[x] = i;
            r[x] = i;
        }
        auto get = [&](int start) 
        { //求start开始符合条件2的子字符串
            int end = r[s[start] - 'a'];
            for (int j = start + 1; j < end; j++)
            {
                if (l[s[j] - 'a'] < start)
                    return -1;
                end = max(end, r[s[j] - 'a']);
            }
            return end;
        };
        vector<string> res = {""};
        int right = INT_MAX;
        for (int i = 0; i < s.size(); i++)
        {
            if (i == l[s[i] - 'a'])
            { //只找以当前字符在字符串第一次出现的字符开始的子字符串，所以最多找26次
                int r = get(i);
                if (r == -1)
                    continue;
                if (i > right)
                {
                    right = r;
                    res.push_back("");
                }
                right = min(right, r);
                res.back() = s.substr(i, r - i + 1);
            }
        }
        return res;
    }
};
```
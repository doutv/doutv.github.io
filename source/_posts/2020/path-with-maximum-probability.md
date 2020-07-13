---
title: path-with-maximum-probability
date: 2020-07-13 8:59:36
tags:
    - Algorithm
    - Leetcode
    - shortest path
mathjax: true
categories:
    - [Algorithm]
cover: true
---

# 5211. 概率最大的路径

## dfs
我第一反应是用dfs，无奈dfs怎么优化都超时，后来看了评论区才知道，加上`if (curp<1e-4) return;`就可以过了，原理也很简单：只要答案与标准答案的误差不超过 `1e-5` ，就会被视作正确答案，所以如果当前已经走过的路径的成功概率太小的话，可以直接不要这条路径。

```cpp
class Solution {
public:
    vector<pair<int,double>> e[10005];
    bool visited[10005];
    double ans;
    double laste;
    void dfs(int u,int end,double curp)
    {
        if (u==end)
        {
            ans=max(ans,curp);
            return;
        }
        if (curp<1e-5||curp*laste<ans)
            return;
        for (int i=0;i<e[u].size();i++)
        {
            int v=e[u][i].first;
            if (visited[v])
                continue;
            double p=e[u][i].second;
            visited[v]=1;
            dfs(v,end,curp*p);
            visited[v]=0;
        }
    }
    double maxProbability(int n, vector<vector<int>>& edges, vector<double>& s, int start, int end) {
        for (int i=0;i<edges.size();i++)
        {
            if (s[i]>0)
            {
            e[edges[i][0]].push_back(make_pair(edges[i][1],s[i]));
            e[edges[i][1]].push_back(make_pair(edges[i][0],s[i]));
            }
        }
        for (int i=0;i<e[end].size();i++)
            laste=max(laste,e[end][i].second);
        dfs(start,end,1);
        return ans;
    }
};
```

## dijkstra
dfs算法超时后，我立刻就想到了最短路算法：dijkstra。

这题与最短路本质上是一样的，最短路是将每条边加起来，求最小边权和；而这题是将每条边乘起来，求最大边权乘积。
证明方法：
- [证明贪心是正确的](https://leetcode-cn.com/problems/path-with-maximum-probability/solution/zui-duan-lu-suan-fa-ji-zheng-que-xing-de-yan-jin-z/)，每个点一旦被加入集合，之后就不能出现一条更短的路径了。

但是我一开始的dijkstra写错了，优先队列中的节点按照外部数组`dis`从大到小排列，这样会导致一个问题：
- 队列里的元素的优先级是由外界的`dis`变化决定的，priority_queue就无法实时保证队列里的元素满足堆性质了。我只要在外面随便改一改`dis`的值，priority_queue就废了。
- https://www.luogu.com.cn/discuss/show/51361

所以priority_queue中存储的应该是`pair<double,int> {p,u}`，优先队列中的元素优先级不依赖于外部元素。

此处不需要重载运算符：
- 优先队列默认的比较类型是：`class Compare = std::less<typename Container::value_type>`。
- `less`会调用类型 T 上的 `operator<` ，除非特化。
- `pair<class T1, class T2>`的小于号`<`会先比较第一个元素`.first`，再比较第二个元素`.second`。
- 所以`priority_queue<pair<double,int>> q`是按`p`从大到小排列。

```cpp
class Solution {
public:
    vector<pair<double,int>> e[10005];
    bool visited[10005];
    double maxProbability(int n, vector<vector<int>>& edges, vector<double>& s, int start, int end) {
        for (int i=0;i<edges.size();i++)
        {
            e[edges[i][0]].push_back({s[i],edges[i][1]});
            e[edges[i][1]].push_back({s[i],edges[i][0]});
        }
        vector<double> dis(n+1,0);
        priority_queue<pair<double,int>> q;
        dis[start]=1;
        q.push({1.0,start});
        while (!q.empty())
        {
            auto [p,u]=q.top();
            q.pop();
            if (visited[u])
                continue;
            visited[u]=1;
            for (auto [p,v]:e[u])
            {
                dis[v]=max(dis[v],dis[u]*p);
                if (!visited[v])
                    q.push({dis[v],v});
            }
        }
        return dis[end];
    }
};
```
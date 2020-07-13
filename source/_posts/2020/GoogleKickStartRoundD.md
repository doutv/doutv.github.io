---
title: Google-Kick-Start-RoundD
top: false
cover: true
toc: true
mathjax: true
tags:
  - Algorithm
  - Competition
categories:
  - - Algorithm
date: 2020-07-13 11:31:52
summary:
---

# Google Kick Start Round D
这次成绩是三次以来最好的啦，1332名，虽然我只做出了第一题，后面三题都是暴力过第一个数据点~

## Record Breaker
送分模拟题，注意一下边界条件即可。
`a[n+1],premax`都初始化为`-1`，这样就不需要特判边界了。

```cpp
int a[200005];
void solve()
{
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    int premax = -1;
    int ans = 0;
    a[n + 1] = -1;
    for (int i = 1; i <= n; i++)
    {
        if (a[i] > premax && a[i] > a[i + 1])
            ++ans;
        premax = max(premax, a[i]);
    }
    cout << ans << endl;
}
```

## Alien Piano
比赛时看这题觉得很复杂，不知道用什么方法来构造这个序列。于是就跳过去了，后面回来把暴力$O(4^N)$写了：

### 暴力
```cpp
int a[15];
int b[15];
int ans;
void dfs(int cur, int n, int res)
{
    if (cur > n)
    {
        int cnt = 0;
        for (int i = 1; i < n; i++)
        {
            if ((a[i] < a[i + 1] && b[i] < b[i + 1]) || (a[i] > a[i + 1] && b[i] > b[i + 1]) || (a[i] == a[i + 1] && b[i] == b[i + 1]))
                continue;
            ++cnt;
        }
        ans = min(ans, cnt);
        return;
    }
    for (int i = 1; i <= 4; i++)
    {
        b[cur] = i;
        dfs(cur + 1, n, res);
    }
}
void solve()
{
    int k;
    cin >> k;
    ans = k;
    for (int i = 1; i <= k; i++)
        scanf("%d", &a[i]);
    dfs(1, k, 0);
    cout << ans << endl;
}
```

看完题解觉得还好，有两种方法：
### 贪心
- 直接找长度大于$4$的连续上升/下降子序列，每有一个这样的子序列，就要打破一次规则
- 我当时怎么没想到呢？

```cpp
void solve()
{
    int k;
    cin>>k;
    for (int i=1;i<=k;i++)
        cin>>a[i];
    int ans = 0,cnt1 = 1,cnt2 = 1;
    for(int i = 2; i <= k; i++){
        if( a[i - 1] < a[i] ){
            cnt1++;
            cnt2 = 1;
        }
        else if(a[i - 1] > a[i]){
            cnt2++;
            cnt1 = 1;
        }
        if(cnt1 > 4 || cnt2 > 4){
            cnt1 = cnt2 = 1;
            ans++;
        }
    }
    cout << ans << '\n';
}
```

### dp
- $dp(i,j)$表示前$i$个note，其中第$i$个note用$j$来表示的最小规则打破数。
- $$ dp(i,j)=\min{dp(i-1,j')+P(i,j',j)|1 \leq j' \leq 4} $$
- $P(i,j',j)$表示：第$i-1$个note用$j'$来表示，第$i$个note用$j$来表示，需要打破的规则数。

```cpp
void solve()
{
    int k;
    cin >> k;
    for (int i = 1; i <= k; i++)
        scanf("%d", &a[i]);
    for (int j = 1; j <= 4; j++)
        dp[1][j] = 0;
    for (int i = 2; i <= k; i++)
        for (int j = 1; j <= 4; j++)
        {
            dp[i][j] = INT32_MAX;
            for (int k = 1; k <= 4; k++)
            {
                int p;
                if ((k < j && a[i - 1] < a[i]) || (k > j && a[i - 1] > a[i]) || (k == j && a[i - 1] == a[i]))
                    p = 0;
                else
                    p = 1;
                dp[i][j] = min(dp[i][j], dp[i - 1][k] + p);
            }
        }
    int ans = k;
    for (int j = 1; j <= 4; j++)
        ans = min(ans, dp[k][j]);
    cout << ans << endl;
}
```

## Beauty of tree
这题一开始没审清楚题目，`travels up the tree`应该是向上找父节点，我以为是向上找第`A`层的随便一个节点都行，白做了大半个小时😓，后来只能用暴力咯。

### 暴力
- 要用`printf("%f")`来输出`double`类型，用`cout`似乎精度有点问题，过不了第一个测试集。
- 遍历每一对选择的节点`(i,j)`时，`i`和`j`都要`1 to n`

```cpp
int f[105];
bool visited[105];
void solve()
{
    int n, a, b;
    cin >> n >> a >> b;
    for (int i = 1; i <= n; i++)
        f[i] = i;
    for (int i = 2; i <= n; i++)
    {
        int x;
        scanf("%d", &x);
        f[i] = x;
    }
    ll ans = 0;
    int tot = 0;
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        // choose node i,j
        {
            ++tot;
            int x = i;
            int y = j;
            for (int k = 1; k <= n; k++)
                visited[k] = 0;
            if (x == y)
                ++ans;
            else
                ans += 2;
            visited[x] = 1;
            visited[y] = 1;
            int cnt = 0;
            while (x != f[x])
            {
                x = f[x];
                ++cnt;
                if (cnt % a == 0 && !visited[x])
                {
                    visited[x] = 1;
                    ++ans;
                }
            }
            cnt = 0;
            while (y != f[y])
            {
                y = f[y];
                ++cnt;
                if (cnt % b == 0 && !visited[y])
                {
                    visited[y] = 1;
                    ++ans;
                }
            }
        }
    }
    printf("%f\n", (double)ans / tot);
}
```

### dfs
原来dfs还有这个用处啊，学到了学到了。

- `P(being visited by either Amadea or Bilva) = P(visited by Amadea) + P(visited by Bilva) - (P(visited by Amadea)*P(visited by Bilva))`
- `P(visited by Amadea) = visited times / total number of nodes`
- 所以只需要算每个节点访问过的次数即可。
- 首先，每个节点至少访问过一次，因为至少会被选到一次。
- 其次，越靠近叶子节点，访问次数会越多。
- 每遍历到一个点，就给它的第`A`/`B`个父节点加上它自己的访问次数。
- 用一个辅助数组`path`来记录从根节点`1`到当前节点`u`经过的所有节点

```cpp
class Solution
{
    vector<int> a_visit, b_visit;
    vector<vector<int>> children;
    vector<int> path;
    int n, a, b;

    void dfs(int u)
    {
        path.emplace_back(u);
        a_visit[u] = 1;
        b_visit[u] = 1;
        for (int v : children[u])
            dfs(v);
        path.pop_back();
        if (path.size() >= a)
            a_visit[path[path.size() - a]] += a_visit[u];
        if (path.size() >= b)
            b_visit[path[path.size() - b]] += b_visit[u];
    }

public:
    void solve()
    {
        cin >> n >> a >> b;
        children = vector<vector<int>>(n + 1);
        a_visit = vector<int>(n + 1);
        b_visit = vector<int>(n + 1);
        for (int i = 2; i <= n; i++)
        {
            int x;
            scanf("%d", &x);
            children[x].push_back(i);
        }
        dfs(1);
        double ans = 0;
        for (int i = 1; i <= n; i++)
        {
            double pa = (double)a_visit[i] / n;
            double pb = (double)b_visit[i] / n;
            ans += pa + pb - pa * pb;
        }
        printf("%f\n", ans);
    }
};
```

## Locked Doors
做到这题的时候没时间了，只剩下半小时不到，在最后一分钟的时候才过了第一个样例😄。

### 模拟
该题的模拟并不难想，只需用两个指针`l`和`r`来维护左右锁住的门的下标即可，复杂度为$O(NQ)$

- $N$个房间只有$N-1$个门，注意数组大小。
- 边界条件：`a[0],a[n]=INT_MAX`，当一侧的门全部打开后，只能打开另一侧的门。
- 画图来帮助理解门和房间的对应关系


```cpp
void solve()
{
    int n, q;
    cin >> n >> q;
    for (int i = 1; i <= n - 1; i++)
        scanf("%d", &a[i]);
    a[0] = INT32_MAX, a[n] = INT32_MAX;
    while (q--)
    {
        int s, k;
        scanf("%d %d", &s, &k);
        int l = s - 1, r = s;
        for (int i = 2; i <= k; i++)
        {
            if (a[l] < a[r])
            {
                --l;
                s = l + 1;
            }
            else
            {
                ++r;
                s = r;
            }
        }
        printf("%d ", s);
    }
}
```


## 总结
做了几次Google的比赛了，题风跟别处还是有很大不同的：
- 注重边界条件的考察
- 不会告诉你哪个数据点错了，只能自己想程序中不完善的地方
- 分析题目的能力 > 积累算法的多少

---
title: AtCoder ABC209题解
date: 2021/07/11 11:41:57
categories: 
- 题解
tags: 
- 题解
- Atcoder
---

# AtCoder Beginner Contest 209题解

传送门： [AtCoder Beginner Contest 209](https://atcoder.jp/contests/abc209)

<!--more-->

## A. Counting

题意：输出$[a,b]$之间有多少个数。

思路：签到题，注意可能$a>b$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a, b; cin >> a >> b;
    cout << max(0, b - a + 1) << endl;
    return 0;
}
```

## B. Can you buy them all? 

题意：有$N$件商品，偶数天的商品会便宜$1$元，给了$X$元，求能不能把$N$件商品买完。

思路：签到题，直接模拟。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, x; cin >> n >> x;
    vector<int> a(n + 1);
    int sum = 0;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        if (i & 1) sum += a[i];
        else sum += a[i] - 1;
    }
    if (sum > x) cout << "No" << endl;
    else cout << "Yes" << endl;
    return 0;
}
```

## C. Not Equal

题意：找到有多少序列满足以下条件：$1 \le A_i \le C_i (1 \le i \le N)$，$A_i \ne A_j (1 \le i < j \le N)$。

思路：将$C[i]$从小到大排序，则第$i$个可以选的数字个数为$C[i] - (i - 1)$，乘上答案即可。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const ll MOD = 1e9+7;
const int MAX = 2e5+50;
int C[MAX], n;

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> C[i];
    sort(C + 1, C + n + 1);
    ll res = 1;
    for (int i = 1; i <= n; i++) {
        res = res * (C[i] - (i - 1)) % MOD;
    }
    cout << res << endl;
    return 0;
}
```

## D. Collision

题意：给了一棵树，每次询问两个点会在边上相遇还是点上相遇。

思路：如果两个点最短距离为奇数，会在边上相遇，否则会在点上相遇，可以利用$lca$求两点在树上的最短距离。还看到一种神奇的写法，可以利用二分图染色，如果两点颜色不同在边上相遇，否则在点上相遇。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1e5+50;
vector<int> es[MAX];
int fa[MAX], deep[MAX], sz[MAX], son[MAX];
int top[MAX];

void dfs1(int u, int pre, int dep) {
    fa[u] = pre;
    deep[u] = dep;
    sz[u] = 1;
    for (auto& v: es[u]) {
        if (v == pre) continue;
        dfs1(v, u, dep + 1);
        sz[u] += sz[v];
        if (sz[v] > sz[son[u]]) son[u] = v;
    }
}

void dfs2(int u, int utop) {
    top[u] = utop;
    if (son[u]) dfs2(son[u], utop);
    for (auto& v: es[u]) {
        if (v == fa[u] || v == son[u]) continue;
        dfs2(v, v);
    }
}

int LCA(int u, int v) {
    while (top[u] != top[v]) {
        if (deep[top[u]] < deep[top[v]]) swap(u, v);
        u = fa[top[u]];
    }
    if (deep[u] > deep[v]) swap(u, v);
    return u;
}

int main() {
    int n, q; cin >> n >> q;
    for (int i = 1; i < n; i++) {
        int u, v; cin >> u >> v;
        es[u].push_back(v);
        es[v].push_back(u);
    }
    dfs1(1, 0, 1);
    dfs2(1, 1);
    while (q--) {
        int u, v; cin >> u >> v;
        int lca = LCA(u, v);
        int dis = deep[u] - deep[lca] + deep[v] - deep[lca];
        if (dis & 1) cout << "Road" << endl;
        else cout << "Town" << endl;
    }
    return 0;
}
```

## E. Shiritori

题意：博弈游戏，给了$N$个单词，类似于单词接龙的游戏，每个人说的单词必须在这$N$个单词之中且前三个字母要跟上一个人说的单词的后三个字母相同，直到某人接不来就输了，可能会平局。

思路：可以把每个单词的前三个字母和后三个字母看成是有向图上的点，并将他们连一条有向边。问题转化为两个人轮流在有向图上移动一个点直到点不能动时结束。我们可以给每个点定义三个状态：平局，必败，必胜。如果一个点没有任何出边，那么这个点是必败的；如果一个点的后继节点有必败态，那么这个节点是必胜的；如果一个点的后继节点全是必胜态，那么这个节点是必败的；其他情况为平局。可以利用类似拓扑排序倒着更新每个点的状态，从而将复杂度降为$O(N)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e5+50;
int n, idx, out[MAX], dp[MAX];
vector<int> es1[MAX], es2[MAX];
string s[MAX];
map<string, int> mp;
int ans[MAX];
queue<int> q;

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> s[i];
        string su = s[i].substr(0, 3);
        string sv = s[i].substr(s[i].length() - 3);
        if (!mp[su]) mp[su] = ++idx;
        if (!mp[sv]) mp[sv] = ++idx;
        int u = mp[su], v = mp[sv];
        es1[u].emplace_back(v);
        es2[v].emplace_back(u);
        ++out[u];
    }
    memset(dp, -1, sizeof(dp));
    for (int i = 1; i <= idx; i++) {
        if (!out[i]) {
            dp[i] = 0;
            q.push(i);
        }
    }
    while (q.size()) {
        int u = q.front(); q.pop();
        for (auto& v: es2[u]) {
            --out[v];
            if (dp[u] == 0 && dp[v] == -1) {
                dp[v] = 1;
                q.push(v);
            }
            if (dp[u] == 1 && dp[v] == -1 && !out[v]) {
                dp[v] = 0;
                q.push(v);
            }
        }
    }
    //-1:draw, 0:lose, 1:win
    for (int i = 1; i <= n; i++) {
        int now = dp[mp[s[i].substr(s[i].length() - 3)]];
        if (now == 0) {
            cout << "Takahashi\n";
        } else if (now == 1) {
            cout << "Aoki\n";
        } else {
            cout << "Draw\n";
        }
    }
    return 0;
}
```

## F. Deforestation

题意：你需要找出一种排列，按照排列的顺序去砍树，使得花费最小，砍一棵树的花费是当前这棵树及左右两棵树的高度总和。

思路：

```cpp

```

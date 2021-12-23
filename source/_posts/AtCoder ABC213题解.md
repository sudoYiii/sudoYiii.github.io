---
title: AtCoder ABC213题解
date: 2021/08/09 09:19:00
categories: 
- 题解
tags: 
- 题解
- Atcoder
---

# AtCoder Beginner Contest 213题解

传送门：[AtCoder Beginner Contest 213](https://atcoder.jp/contests/abc213) 

<!--more-->

## A. Bitwise Exclusive Or  

题意：输出两个数异或的结果。

思路：无。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a, b; scanf("%d%d", &a, &b);
    printf("%d\n", a ^ b);
    return 0;
}
```

## B. Booby Prize   

题意：输出第二大的数的下标。

思路：无。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e5+50;
struct Node {
    int id, a;
    bool operator < (const Node& p) const {
        return a > p.a;
    }
}node[MAX];
int n;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &node[i].a);
        node[i].id = i;
    }
    sort(node + 1, node + n + 1);
    printf("%d\n", node[2].id);
    return 0;
}
```

## C. Reorder Cards  

题意：如果某一列或某一行没有数字，就将那列或行删除，直到不能删为止，求最后所有数的坐标。

思路：显然，我们可以将行和列分开考虑，对于某一个数字，它最终横坐标即**（原始横坐标 - 前面不同横坐标数量）**，纵坐标同理可得。那么就可以离线后将所有坐标进行排序和离散化，然后二分进行查找所在位置。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1e5+50;
struct Node {
    int x, y;
}node[MAX];
int x[MAX], y[MAX], h, w, n;

int main() {
    scanf("%d%d%d", &h, &w, &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d", &node[i].x, &node[i].y);
        x[i] = node[i].x;
        y[i] = node[i].y;
    }
    sort(x + 1, x + n + 1);
    sort(y + 1, y + n + 1);
    int nx = unique(x + 1, x + n + 1) - (x + 1);
    int ny = unique(y + 1, y + n + 1) - (y + 1);
    for (int i = 1; i <= n; i++) {
        int px = lower_bound(x + 1, x + nx + 1, node[i].x) - x;
        int py = lower_bound(y + 1, y + ny + 1, node[i].y) - y;
        printf("%d %d\n", px, py);
    }
    return 0;
}
```

## D. Takahashi Tour  

题意：输出一棵树的欧拉序，并要求每次从编号最小的儿子遍历。

思路：先对所有边进行从小到大排序，再利用$dfs$打印出欧拉序即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e5+50;
vector<int> es[MAX];
int n;

void dfs(int u, int fa) {
    printf("%d ", u);
    for (auto& v: es[u]) {
        if (v == fa) continue;
        dfs(v, u);
        printf("%d ", u);
    }
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i < n; i++) {
        int u, v; scanf("%d%d", &u, &v);
        es[u].push_back(v);
        es[v].push_back(u);
    }
    for (int i = 1; i <= n; i++) sort(es[i].begin(), es[i].end());
    dfs(1, 0);
    return 0;
}
```

## E. Stronger Takahashi  

题意：有一个$H \times W$的网格，某些点上有障碍，可以花费$1$元去除一个$2 \times 2$的障碍，求从左上角到右下角的最少花费。

思路：对于一个点，在它四个方向的一个距离以内，且没有障碍，花费为$0$；否则以这个点两个距离内，且有障碍的点，花费为$1$。此时可以根据这种关系建立有向边，然后跑$1-H \times W$的最短路，最小花费即为最短路长度。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 505;
struct Edge {
    int v, w, next;
}es[MAX * MAX * 30];
int head[MAX * MAX], k, h, w;
int d[MAX * MAX], id[MAX][MAX], idx;
char s[MAX][MAX];
bool vis[MAX * MAX];
int f[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };

void add(int u, int v, int w) {
    es[++k].next = head[u]; es[k].v = v; es[k].w = w; head[u] = k;
}

int dij() {
    memset(d, 0x3f, sizeof(d));
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> q;
    q.push(make_pair(0, 1));
    d[1] = 0;
    while (q.size()) {
        int u = q.top().second; q.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (int i = head[u]; i; i = es[i].next) {
            int v = es[i].v;
            if (!vis[v] && d[v] > d[u] + es[i].w) {
                d[v] = d[u] + es[i].w;
                q.push(make_pair(d[v], v));
            }
        }
    }
    return d[id[h][w]];
}

int main() {
    scanf("%d%d", &h, &w);
    for (int i = 1; i <= h; i++) scanf("%s", s[i] + 1);
    for (int i = 1; i <= h; i++) {
        for (int j = 1; j <= w; j++) {
            id[i][j] = ++idx;
        }
    }
    for (int i = 1; i <= h; i++) {
        for (int j = 1; j <= w; j++) {
            for (int t = 0; t < 4; t++) {
                int nx = i + f[t][0];
                int ny = j + f[t][1];
                if (nx >= 1 && nx <= h && ny >= 1 && ny <= w) {
                    if (s[nx][ny] == '.') add(id[i][j], id[nx][ny], 0);
                }
            }
            for (int x = -2; x <= 2; x++) {
                for (int y = -2; y <= 2; y++) {
                    if (abs(x) == 2 && abs(y) == 2) continue;
                    int nx = i + x, ny = j + y;
                    if (nx >= 1 && nx <= h && ny >= 1 && ny <= w) {
                        add(id[i][j], id[nx][ny], 1);
                    }
                }
            }
        }
    }
    printf("%d\n", dij());
    return 0;
}
```

## F. Common Prefixes  

题意：

思路：

```cpp

```

## G. Connectivity 2 

题意：

思路：

```cpp

```

## H. Stroll 

题意：

思路：

```cpp

```
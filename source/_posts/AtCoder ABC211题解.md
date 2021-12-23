---
title: AtCoder ABC211题解
date: 2021/07/25 11:35:00
categories: 
- 题解
tags: 
- 题解
- Atcoder
---

# AtCoder Beginner Contest 211题解

传送门：[AtCoder Beginner Contest 211](https://atcoder.jp/contests/abc211) 

<!--more-->

## A. Blood Pressure 

题意：给了$A$和$B$，算$C$。$C = \frac{A - B}{3} + B$。

思路：暴力，注意浮点数。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a, b; scanf("%d%d", &a, &b);
    printf("%.10f\n", (a - b) / 3.0 + (double)b);
    return 0;
}
```

## B. Cycle Hit  

题意：给了四个字符串，判断它们是否为$H, 2B, 3B, HR$四个字符串。

思路：暴力。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    set<string> st;
    st.insert("H");
    st.insert("2B");
    st.insert("3B");
    st.insert("HR");
    set<string> st2;
    for (int i = 0; i < 4; i++) {
        string s; cin >> s;
        st2.insert(s);
    }
    puts(st == st2 ? "Yes" : "No");
    return 0;
}
```

## C. chokudai 

题意：在一个字符串中按照chokudai的顺序选择8个字母有多少种选法。

思路：$\displaystyle dp[i][j] =  \begin{cases} 1 & ( j = 0) \newline 0  & ( i=0 , 1 \leq j \leq 8)  \newline dp[i-1][j]& ( 1 \leq i \leq N , 1\leq j \leq 8,  S[i] \neq T[j] )   \newline dp[i-1][j] + dp[i-1][j-1]  & (1 \leq i \leq N , 1\leq j \leq 8,  S[i] = T[j])\end{cases}$。

最后的答案位$dp[N][8]$。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const ll MOD = 1e9+7;
const int MAX = 1e5+50;
char s[MAX], t[10] = "chokudai";
int n;

int main() {
    scanf("%s", s + 1);
    n = strlen(s + 1);
    map<char, ll> mp;
    for (int i = 1; i <= n; i++) {
        int pos = find(t, t + 8, s[i]) - t;
        if (pos == 8) continue;
        if (s[i] == 'c') ++mp['c'];
        else mp[s[i]] = (mp[s[i]] + mp[t[pos - 1]]) % MOD;
    }
    printf("%lld\n", mp['i'] % MOD);
    return 0;
}
```

## D. Number of Shortest paths 

题意：从$1$到$n$的最短路有多少条。

思路：经典最短路变形，只需要在求最短路的同时更新$cnt$数组即可，初始化$cnt[1] = 1$。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const ll MOD = 1e9+7;
const int MAX = 2e5+50;
struct Edge {
    int v, next;
}es[MAX<<1];
int head[MAX], k, n, m;
int dis[MAX];
ll cnt[MAX];
bool vis[MAX];

void add(int u, int v) {
    es[++k].next = head[u]; es[k].v = v; head[u] = k;
    es[++k].next = head[v]; es[k].v = u; head[v] = k;
}

void dij() {
    memset(dis, 0x3f, sizeof(dis));
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> q;
    q.push(make_pair(0, 1));
    dis[1] = 0;
    cnt[1] = 1;
    while (q.size()) {
        int u = q.top().second; q.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (int i = head[u]; i; i = es[i].next) {
            int v = es[i].v;
            if (!vis[v] && dis[v] > dis[u] + 1) {
                dis[v] = dis[u] + 1;
                cnt[v] = cnt[u];
                q.push(make_pair(dis[v], v));
            } else if (dis[v] == dis[u] + 1) {
                cnt[v] = (cnt[v] + cnt[u]) % MOD;
            }
        }
    }
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; i++) {
        int u, v; scanf("%d%d", &u, &v);
        add(u, v);
    }
    dij();
    printf("%lld\n", cnt[n]);
    return 0;
}
```

## E. Red Polyomino 

题意：有一个$n×n$的地图，连续染$k$个方块，有多少种方式。

思路：因为$n$最多只有$8$，所以方法很多，如果直接进行$dfs$不带剪枝会超时，有一种比较好理解的做法是将每次$dfs$的图利用$set$存起来，如果下次遇到直接返回。再优化一下可以将$set$里放一个$unsigned \ long \ long$型的数字，代表整张地图，因为最多就$64$位刚好存放的下。最快的做法是将该点的所有状态找完后将该点设置为$\#$，再在新图上$dfs$。

```cpp
#include <bits/stdc++.h>
using namespace std;

set<vector<string>> st;
vector<string> s;
int f[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };
int n, k, res;

void dfs(int now) {
    if (st.find(s) != st.end()) return;
    st.insert(s);
    if (now == k) return (void)(++res);
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (s[i][j] != '.') continue;
            bool flag = false;
            for (int k = 0; k < 4 && !flag; k++) {
                int nx = i + f[k][0];
                int ny = j + f[k][1];
                if (nx >= 0 && nx < n && ny >= 0 && ny < n && s[nx][ny] == '@') flag = true; 
            }
            if (!flag) continue;
            s[i][j] = '@';
            dfs(now + 1);
            s[i][j] = '.';
        }
    }
}

int main() {
    cin >> n >> k;
    s = vector<string>(n);
    for (auto& v: s) cin >> v;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (s[i][j] == '.') {
                s[i][j] = '@';
                dfs(1);
                s[i][j] = '.';
            }
        }
    }
    cout << res << "\n";
    return 0;
}
```

## F. Rectilinear Polygons 

题意：

思路：

```cpp

```

---
title: CFRound 731(Div. 3)题解
date: 2021/07/11 11:38:57
categories: 
- 题解
tags: 
- 题解
- Codeforces
---

# Codeforces Round #731 (Div. 3)题解

传送门：[Codeforces Round #731 (Div. 3) - Codeforces](https://codeforces.com/contest/1547)

<!--more-->

## A. Shortest Path with Obstacle

题意：给了三个点的坐标$A, B, F$，求$A$到$B$不经过$F$的最短距离。

思路：很容易发现，只有当$A, B$在同一行或同一列，且$F$在两点之间才会有影响。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int T; cin >> T;
    while (T--) {
        int xa, ya, xb, yb, xf, yf;
        cin >> xa >> ya >> xb >> yb >> xf >> yf;
        if (xa == xb && xa == xf && min(ya, yb) <= yf && yf <= max(ya, yb)) {
            cout << 2 + abs(ya - yb) << endl;
        } else if (ya == yb && ya == yf && min(xa, xb) <= xf && xf <= max(xa, xb)) {
            cout << 2 + abs(xa - xb) << endl;
        } else {
            cout << abs(xa - xb) + abs(ya - yb) << endl;
        }
    }
    return 0;
}
```

## B. Alphabetical Strings

题意：字符串$s$会按照一定的规则形成：第$i$步选择第$i$个小写字母加入$s$左边或右边。给了字符串$s$，判断是否可以用此规则形成。

思路：可以逆着推，即考虑从第$n$个字母开始删除，利用双指针，如果最后$l>r$则合法，否则不合法。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int T; cin >> T;
    while (T--) {
        string s; cin >> s;
        int n = s.length();
        int l = 0, r = n - 1;
        for (char i = n - 1 + 'a'; i >= 'a'; i--) {
            if (s[l] == i) ++l;
            else if (s[r] == i) --r;
            else break;
        }
        if (l > r) cout << "YES\n";
        else cout << "NO\n";
    }
    return 0;
}
```

## C. Pair Programming

题意：有一份已经有$k$行的文档，两人会在文档上进行操作，并给出了两人的操作序列，他们可以轮流操作，如果为$0$说明新增一行，否则如果为$x$，说明修改第$x$行，判断两人的操作序列是否合法，如果合法，输出他们两人合起来的操作序列，否则输出$-1$。

思路：利用双指针，如果某人当前操作的值小于等于$k$行，那么就不断加入$ans$，如果他们的序列不合法，一定会是两人都无法操作的情况。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int T; cin >> T;
    while (T--) {
        int k, n, m; cin >> k >> n >> m;
        vector<int> a(n), b(m);
        for (auto& v: a) cin >> v;
        for (auto& v: b) cin >> v;
        bool ok = true;
        int p1 = 0, p2 = 0;
        vector<int> ans;
        while (p1 < n || p2 < m) {
            bool f = false;
            while (p1 < n && a[p1] <= k) ans.push_back(a[p1]), k += a[p1] == 0, ++p1, f = true;
            while (p2 < m && b[p2] <= k) ans.push_back(b[p2]), k += b[p2] == 0, ++p2, f = true;
            if (!f) { ok = false; break; }
        }
        if (!ok) cout << "-1\n";
        else {
            for (auto& v: ans) cout << v << " ";
            cout << "\n";
        }
    }
    return 0;
}
```

## D. Co-growing Sequence

题意：称一个序列为成长序列，如果对于所有$a_i (1 \le i \le n - 1)$，有$a_i \: \& \: a_{i+1} = a_i$。现在给了你序列$x$，你需要找到字典序最小的序列$y$，使得$x_i \oplus y_i (1 \le i \le n)$是成长序列。

思路：有一个贪心的策略是尽量只改变后面的。分类讨论情况，当二进制下$x_i$的第$j$位为$0$，那么不管$x_{i+1}$的第$j$位是什么，$\&$之后都为$0$，此时$y_{i+1}$的第$j$位为$0$；如果$x_i$的第$j$位为$1$，且$x_{i+1}$的第$j$位为$0$，那么就要让$y_{i+1}$这一位为$1$， 否则为$0$。最后还要让$x_{i+1}$去异或$y_{i+1}$，否则后面计算会出问题。

```cpp
#include <bits/stdc++.h>
using namespace std;
 
int main() {
    int T; cin >> T;
    while (T--) {
        int n; cin >> n;
        vector<int> x(n), y(n, 0);
        for (auto& v: x) cin >> v;
        for (int i = 0; i < n - 1; i++) {
            if ((x[i] & x[i + 1]) == x[i]) continue;
            else {
                for (int j = 0; j < 30; j++) {
                    if (((x[i] >> j) & 1) == 0) continue;
                    if (((x[i] >> j) & 1) == 1 && ((x[i + 1] >> j) & 1) == 0) {
                        y[i + 1] |= (1 << j);
                    }
                }
                x[i + 1] ^= y[i + 1];
            }
        }
        for (auto& v: y) cout << v << " ";
        cout << "\n";
    }
    return 0;
}
```

## E. Air Conditioners

题意：有$k$个空调，给了你位置和温度，每个位置定义了一个温度，计算公式为$min_{1 \le j \le k}(t_j + \mid a_j - i \mid)$。求每个位置的温度。

思路：该公式即所有空调与某一位置的距离+该空调的温度的最小值。可以分析出，对于任意位置来说，它的温度是来自左边或者右边的空调，那么就可以利用两次$dp​$，一次从左向右，一次从右向左，最后两个方向里取最小的。例如从左向右的$dp​$，我们先初始化$dp​$数组为正无穷，如果该位置本身就有空调，我们可以将它先设置为该空调的温度，$dp​$方程就为$dp[i] = min(dp[i - 1] + 1, dp[i])​$，反之相同。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;
 
const int MAX = 3e5+50;
struct Node {
    int a, t;
}nd[MAX];
int a[MAX];
ll dp[2][MAX], t[MAX];
 
int main() {
    int T; cin >> T;
    while (T--) {
        int n, k; cin >> n >> k;
        fill(dp[0], dp[0] + n + 5, LLONG_MAX - 5);
        fill(dp[1], dp[1] + n + 5, LLONG_MAX - 5);
        fill(t, t + n + 5, LLONG_MAX - 5);
        for (int i = 1; i <= k; i++) cin >> nd[i].a;
        for (int i = 1; i <= k; i++) cin >> nd[i].t, t[nd[i].a] = nd[i].t;
        for (int i = 1; i <= n; i++) {
            dp[0][i] = min(dp[0][i - 1] + 1, t[i]);
        }
        for (int i = n; i >= 1; i--) {
            dp[1][i] = min(dp[1][i + 1] + 1, t[i]);
        }
        for (int i = 1; i <= n; i++) cout << min(dp[0][i], dp[1][i]) << " ";
        cout << "\n";
    }
    return 0;
}
```

## F. Array Stabilization (GCD version)

题意：给了一个数组$a$，可以执行以下操作任意次：$a_i = \gcd(a_i, a_{(i + 1) \mod n})(0 \le i \le n - 1)$，最少多少次可以使数组所有数相同。

思路：二分+线段树。先来看下对于$x_1$经过几次$gcd$会变成什么，第一次：$gcd(x_1,x_2)$，第二次：$gcd(x_1, gcd(x_2, x_3)) = gcd(x_1, x_2, x_3)$，通过归纳法可以发现经过$t$步后$x_1$会变成$gcd(x_1,x_2, ..., x_{t+1})$，此时可以通过枚举要多少步可以使所有所有$x_i$相同，求区间$gcd$可以利用线段树，由于最多可能需要$n$步，所以还需要利用二分。后面几个数求区间$gcd$可能会不方便，即要查两次$gcd$，我们可以先复制一遍$x$数组，这样就只需查询一遍。

```cpp
#include <bits/stdc++.h>
#define ls k<<1
#define rs k<<1|1
#define lson k<<1, l, mid
#define rson k<<1|1, mid+1, r
using namespace std;

const int MAX = 4e5+50;
struct Node {
    int l, r, g;
}tree[MAX<<2];
int a[MAX], n;

void build(int k, int l, int r) {
    tree[k].l = l; tree[k].r = r;
    if (l == r) {
        tree[k].g = a[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(lson);
    build(rson);
    tree[k].g = __gcd(tree[ls].g, tree[rs].g);
}

int query(int k, int l, int r) {
    if (tree[k].l == l && tree[k].r == r) return tree[k].g;
    int mid = (tree[k].l + tree[k].r) >> 1;
    if (r <= mid) return query(ls, l, r);
    else if (l > mid) return query(rs, l, r);
    else return __gcd(query(lson), query(rson));
}

bool check(int x) {
    int val = query(1, 1, x);
    for (int i = 2; i <= n; i++) {
        if (val != query(1, i, i + x - 1)) return false;
    }
    return true;
}

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
            a[i + n] = a[i];
        }
        bool flag = true;
        for (int i = 2; i <= n; i++) {
            if (a[i] != a[i - 1]) {
                flag = false;
                break;
            }
        }
        if (flag) {
            puts("0");
            continue;
        }
        build(1, 1, n * 2);
        int l = 1, r = n, res = n;
        while (l <= r) {
            int mid = (l + r) >> 1;
            if (check(mid)) res = mid, r = mid - 1;
            else l = mid + 1;
        }
        printf("%d\n", res - 1);
    }
    return 0;
}
```

## G. How Many Paths?

题意：给了一张有向图，判断$1$号点和每个点的关系：$0$——没有路，$1$——只有$1$条路，$2$——有多条且有限的路，$-1$——无穷的路。

思路：对于$0$很好判断，对于$1$和$2$，可以先用$dfs$去将所有能到达的点设置为$1$，如果途中经过了为$1$的点，则将那个点设置为$2$，然后再将所有$2$的点$dfs$一遍，该点为$2$，则它可以到达的所有点肯定都至少为$2$，对于$-1$，我们可以利用强连通分量缩点，在环上的点以及这个环能到达的所有点一定都是为$-1$。官方题解用了更加巧妙地$dfs$，可以帮助加强对于$dfs$的理解。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 4e5+50;
int dfn[MAX], low[MAX], idx, n, m;
int stk[MAX], top;
int belong[MAX], cnt[MAX], sum;
bool instk[MAX], self[MAX];
vector<int> es[MAX];
int ans[MAX];

void init() {
    for (int i = 0; i < n + 5; i++) es[i].clear();
    memset(ans, 0, sizeof(int) * (n + 5));
    memset(dfn, 0, sizeof(int) * (n + 5));
    memset(low, 0, sizeof(int) * (n + 5));
    memset(cnt, 0, sizeof(int) * (n + 5));
    memset(instk, false, sizeof(bool) * (n + 5));
    memset(self, false, sizeof(bool) * (n + 5));
    idx = sum = top = 0;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++idx;
    stk[++top] = u;
    instk[u] = true;
    for (auto& v: es[u]) {
        if (dfn[v] == 0) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (instk[v]) low[u] = min(low[u], dfn[v]);
    }
    if (dfn[u] == low[u]) {
        int v;
        belong[u] = ++sum;
        instk[u] = false;
        do{
            v = stk[top--];
            belong[v] = sum;
            instk[v] = false;
            ++cnt[sum];
        } while (u != v);
    }
}

void dfs1(int u, int cur) {
    for (auto& v: es[u]) {
        if (ans[v] == 0) {
            ans[v] = cur;
            dfs1(v, cur);
        } else if (ans[v] == 1) {
            ans[v] = 2;
            dfs1(v, 2);
        }
    }
}

void dfs2(int u) {
    for (auto& v: es[u]) {
        if (ans[v] == -1) continue;
        ans[v] = -1;
        dfs2(v);
    }
}

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d%d", &n, &m);
        init();
        for (int i = 1; i <= m; i++) {
            int u, v; scanf("%d%d", &u, &v);
            if (u == v) {
                self[u] = true;
                continue;
            }
            es[u].push_back(v);
        }
        ans[1] = 1;
        dfs1(1, 1);
        for (int i = 1; i <= n; i++) if (ans[i] == 2) dfs1(i, 2);
        tarjan(1);
        for (int i = 1; i <= n; i++) {
            if (!dfn[i]) continue;
            if ((self[i] || cnt[belong[i]] > 1) && ans[i] != -1) ans[i] = -1, dfs2(i);
        }
        for (int i = 1; i <= n; i++) printf("%d%c", ans[i], " \n"[i == n]);
    }
    return 0;
}
```






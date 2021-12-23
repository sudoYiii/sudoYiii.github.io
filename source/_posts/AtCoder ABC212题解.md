---
title: AtCoder ABC212题解
date: 2021/08/02 09:07:00
categories: 
- 题解
tags: 
- 题解
- Atcoder
---

# AtCoder Beginner Contest 212题解

传送门：[AtCoder Beginner Contest 212](https://atcoder.jp/contests/abc212) 

<!--more-->

## A. Alloy 

题意：根据题目给的三个条件输出。

思路：模拟。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a, b; scanf("%d%d", &a, &b);
    if (a > 0 && b == 0) puts("Gold");
    else if (a == 0 && b > 0) puts("Silver");
    else puts("Alloy");
    return 0;
}
```

## B. Weak Password  

题意：如果一个密码的所有位数相同或对于所有$i(1 \le i \le 3)$使得$a_i + 1 = a_{i + 1}$为弱密码，否则为强密码。

思路：模拟。

```cpp
#include <bits/stdc++.h>
using namespace std;

bool check(int a[]) {
    for (int i = 2; i <= 4; i++) {
        if (a[i] != (a[i - 1] + 1) % 10) return false;
    }
    return true;
}

int main() {
    int a[5];
    for (int i = 1; i <= 4; i++) scanf("%1d", &a[i]);
    if (a[1] == a[2] && a[1] == a[3] && a[1] == a[4] || check(a)) puts("Weak");
    else puts("Strong");
    return 0;
}
```

## C. Min Difference 

题意：给定两个数组，每个数组寻找一个数使得两数之差的绝对值最小。

思路：很明显差的绝对值最小即要求两数越接近，那么可以使用二分，先将任意数组进行排序，枚举另一数组的每一个数，此时可以二分寻找已排序数组的最接近的值。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int MAX = 2e5+50;
ll a[MAX], b[MAX];
int n, m;

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%lld", &a[i]);
    for (int i = 1; i <= m; i++) scanf("%lld", &b[i]);
    sort(a + 1, a + n + 1);
    a[n + 1] = INT_MAX;
    a[0] = -INT_MAX;
    ll res = LLONG_MAX;
    for (int i = 1; i <= m; i++) {
        int pos = upper_bound(a + 1, a + n + 1, b[i]) - a;
        res = min(res, abs(b[i] - a[pos]));
        res = min(res, abs(b[i] - a[pos - 1]));
    }
    printf("%lld\n", res);
    return 0;
}
```

## D. Querying Multiset 

题意：有三个操作，往背包里放入一个数；将背包里的数全部加上$x$；寻找背包里最小的数取出并输出。

思路：利用线段树，第一个操作相当于找线段树中空的位置进行放入，可以利用一个$cnt$保存当前节点是否有空。第二个操作即普通的线段树区间加法，不过这里利用$pushdown$将标记下方，这样不用考虑有哪些区间，只需要考虑左子树或右子树有无数字即可。第三个操作是去线段树中寻找最小值，可以在线段树上进行二分，寻找完后要将节点初始化。

```cpp
#include <bits/stdc++.h>
#define ls k<<1
#define rs k<<1|1
#define lson k<<1, l, mid
#define rson k<<1|1, mid+1, r
#define ll long long
using namespace std;

const int MAX = 2e5+50;
struct Node {
    int l, r, cnt;
    ll mi, lazy;
}tree[MAX << 2];

void pushup(int k) {
    tree[k].cnt = tree[ls].cnt + tree[rs].cnt;
    tree[k].mi = min(tree[ls].mi, tree[rs].mi);
}

void pushdown(int k) {
    if (!tree[k].lazy) return;
    if (tree[ls].mi != LLONG_MAX) {
        tree[ls].mi += tree[k].lazy;
        tree[ls].lazy += tree[k].lazy;
    }
    if (tree[rs].mi != LLONG_MAX) {
        tree[rs].mi += tree[k].lazy;
        tree[rs].lazy += tree[k].lazy;
    }
    tree[k].lazy = 0;
}

void build(int k, int l, int r) {
    tree[k].l = l; tree[k].r = r;
    tree[k].mi = LLONG_MAX;
    tree[k].lazy = 0;
    if (l == r) {
        tree[k].cnt = 1;
        return;
    }
    int mid = (l + r) >> 1;
    build(lson);
    build(rson);
    pushup(k);
}

void modify1(int k, ll x) {
    if (tree[k].l == tree[k].r) {
        tree[k].mi = x;
        tree[k].cnt = 0;
        return;
    }
    pushdown(k);
    int mid = (tree[k].l + tree[k].r) >> 1;
    if (tree[ls].cnt) modify1(ls, x);
    else modify1(rs, x);
    pushup(k);
}

ll modify2(int k) {
    if (tree[k].l == tree[k].r) {
        ll x = tree[k].mi;
        tree[k].mi = LLONG_MAX;
        tree[k].cnt = 1;
        return x;
    }
    pushdown(k);
    int mid = (tree[k].l + tree[k].r) >> 1;
    ll ans;
    if (tree[ls].mi <= tree[rs].mi) ans = modify2(ls);
    else ans = modify2(rs);
    pushup(k);
    return ans;
}

int main() {
    int Q; scanf("%d", &Q);
    build(1, 1, Q);
    for (int i = 1; i <= Q; i++) {
        int op; scanf("%d", &op);
        if (op == 1) {
            ll x; scanf("%lld", &x);
            modify1(1, x);
        } else if (op == 2) {
            ll x; scanf("%lld", &x);
            if (tree[1].mi != LLONG_MAX) {
                tree[1].mi += x;
                tree[1].lazy += x;
            }
        } else {
            printf("%lld\n", modify2(1));
        }
    }
    return 0;
}
```

## E. Safety Journey 

题意：有一个无向图，图中有一些边不存在，即给了一个补图。求从城市$1$出发经过$k$天回到城市$1$的方案数。

思路：因为图中点可以无限制的经过，且$N$只有$5000$，可以利用$dp$。$dp[i][j]$表示第$i$天到第$j$个城市的方案数，如果图是一张完全图，$dp[i][j] = \sum_{k = 1}^{n} dp[i - 1][k] - dp[i - 1][j]$，即上一天所有城市除去自身的方案数的累加。假设现在与$j$不相连的点为$p_1, p_2, ... p_t$，那么$dp[i][j] = \sum_{k = 1}^{n} dp[i - 1][k] - dp[i - 1][j] - \sum_{k = 1}^{t} dp[i - 1][p[k]]$，即减去所有与它不相连的城市的方案数，因为$k$最多只有$5000$，所以完全可以暴力去减。最后答案为$dp[k][1]$。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const ll MOD = 998244353;
const int MAX = 5e3+50;
int dp[MAX][MAX], n, m, k;
vector<int> es[MAX];

int main() {
    scanf("%d%d%d", &n, &m, &k);
    for (int i = 1; i <= m; i++) {
        int u, v; scanf("%d%d", &u, &v);
        es[u].push_back(v);
        es[v].push_back(u);
    }
    for (int i = 1; i <= n; i++) es[i].push_back(i);
    dp[0][1] = 1;
    for (int i = 1; i <= k; i++) {
        int sum = 0;
        for (int j = 1; j <= n; j++) sum = ((ll)sum + (ll)dp[i - 1][j]) % MOD;
        for (int j = 1; j <= n; j++) {
            int tmp = sum;
            for (auto& v: es[j]) tmp = ((ll)tmp - (ll)dp[i - 1][v] + MOD) % MOD;
            dp[i][j] = tmp;
        }
    }
    printf("%d\n", dp[k][1]);
    return 0;
}
```

## F. Greedy Takahashi 

题意：

思路：

```cpp

```

## G. Power Pair

题意：

思路：

```cpp

```

## H. Nim Counting

题意：

思路：

```cpp

```
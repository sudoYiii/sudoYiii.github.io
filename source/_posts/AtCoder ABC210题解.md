---
title: AtCoder ABC210题解
date: 2021/07/19 10:29:00
categories: 
- 题解
tags: 
- 题解
- Atcoder
---

# AtCoder Beginner Contest 210题解

传送门：[AtCoder Beginner Contest 210](https://atcoder.jp/contests/abc210) 

<!--more-->

## A. Cabbages

题意：签到题。

思路：模拟。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, a, x, y; scanf("%d%d%d%d", &n, &a, &x, &y);
    printf("%d\n", min(n, a) * x + max(0, n - a) * y);
    return 0;
}
```

## B. Bouzu Mekuri 

题意：从最左边开始取数，谁先取到$1$就输。

思路：模拟。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n; cin >> n;
    string s; cin >> s;
    for (int i = 0; i < s.length(); i++) {
        if (s[i] == '1') {
            if (i & 1) {
                cout << "Aoki" << endl;
                return 0;
            } else {
                cout << "Takahashi" << endl;
                return 0;
            }
        }
    }
    return 0;
}
```

## C. Colorful Candies

题意：找到一个连续且大小为$k$的子数组，里面包含不同数的个数最多。

思路：利用$map$存所有数，然后类似滑窗维护不同数个数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 3e5+50;
map<int, int> mp;
int c[MAX];

int main() {
    int n, k; scanf("%d%d", &n, &k);
    for (int i = 1; i <= n; i++) scanf("%d", &c[i]);
    int ans = 0;
    for (int i = 1; i <= k; i++) {
        if (mp[c[i]] == 0) ++ans;
        ++mp[c[i]];
    }
    int tmp = ans;
    for (int i = k + 1; i <= n; i++) {
        --mp[c[i - k]];
        if (mp[c[i - k]] == 0) --tmp;
        if (mp[c[i]] == 0) ++tmp;
        ++mp[c[i]];
        ans = max(ans, tmp);
    }
    printf("%d\n", ans);
    return 0;
}
```

## D. National Railway

题意：要建一条铁路，成本为$a_{x1, y1} + a_{x2, y2} + \mid x1 - x2\mid + \mid y1 - y2 \mid$。找出最小的花费。

思路：考虑动态规划，$dp[i][j][0 \mid 1 \mid 2]$分别表示第$i, j$为起点，中间点及终点的状态。对于起点状态，直接等于$a[i][j]$，对于中间状态，他会从左边或上面的中间或起点最小值$+C$推过来，对于终点状态，就是当前点的中间点状态$+$当前点的$a[i][j]$。需要左右跑两边。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int MAX = 1e3+50;
ll a[MAX][MAX], dp[MAX][MAX][3], C;
int n, m;

ll solve(ll a[MAX][MAX]) {
    memset(dp, 0x3f, sizeof(dp));
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            dp[i][j][0] = a[i][j];
            if (i > 1 || j > 1) {
                dp[i][j][1] = min(dp[i][j][1], min(dp[i - 1][j][0], dp[i - 1][j][1]) + C);
                dp[i][j][1] = min(dp[i][j][1], min(dp[i][j - 1][0], dp[i][j - 1][1]) + C);
                dp[i][j][2] = dp[i][j][1] + a[i][j];
            }
        }
    }
    ll res = LLONG_MAX;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            res = min(res, dp[i][j][2]);
        }
    }
    return res;
}

int main() {
    scanf("%d%d%lld", &n, &m, &C);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            scanf("%lld", &a[i][j]);
        }
    }
    ll res = solve(a);
    for (int i = 1; i <= n; i++) reverse(a[i] + 1, a[i] + m + 1);
    res = min(res, solve(a));
    printf("%lld\n", res);
    return 0;
}
```

## E. 

题意：

思路：

```cpp

```

## F. 

题意：

思路：

```cpp

```

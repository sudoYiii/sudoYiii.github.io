---
title: CFEDU 111(Div. 2)题解
date: 2021/07/15 10:18:21
categories: 
- 题解
tags: 
- 题解
- Codeforces
---

# Educational Codeforces Round 111 (Rated for Div. 2)题解

传送门：[Educational Codeforces Round 111 (Rated for Div. 2)](https://codeforces.com/contest/1550) 

<!--more-->

## A. Find The Array

题意：定义一个漂亮数组$a$为要么$a_i = 1$，要么$a_i - 1$或$a_i - 2$在数组中出现。给了一个数$s$，求一个最小的漂亮数组和为$s$的数组大小。

思路：类似于等差数列求和，当公差为$2$时找到最小的前$n$项和大于等于$s$即为答案。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n; scanf("%d", &n);
        int cnt = 0;
        for (int sum = 0, i = 1; sum < n; sum += i, i += 2) ++cnt;
        printf("%d\n", cnt);
    }
    return 0;
}
```

## B. Maximum Cost Deletion

题意：每次删除一个连续子串，要求子串包含相同字符，直到删完。删除一个子串会得到$a×l+b$分，求最后能得到的最大值。

思路：对于$a × l$，不管取多长最后总和一定是$a × n$($n$为串总长度)，对$b$进行分类讨论，如果$b \ge 0$，那么我们就要让$b$越多越好，所以每次取子串长度为$1$；反之，让$b$越少，我们就要找最少的$0$或$1$的连续串个数即可。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

int s[105];

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n, a, b; scanf("%d%d%d", &n, &a, &b);
        for (int i = 1; i <= n; i++) scanf("%1d", &s[i]);
        if (b >= 0) printf("%d\n", n * (a + b));
        else {
            s[0] = -1;
            int cnt[2] = { 0, 0 };
            for (int i = 1; i <= n; i++) {
                if (s[i] != s[i - 1]) ++cnt[s[i]];
            }
            printf("%d\n", n * a + (min(cnt[0], cnt[1]) + 1) * b);
        }
    }
    return 0;
}
```

## C. Manhattan Subarrays

题意：定义$d(p, q)$为点$p$和点$q$的曼哈顿距离，对于三个点$p, q, r$如果满足$d(p, r) = d(p, q) + d(q, r)$就是坏的三元组。给了一个数组，找到有多少连续的子数组不包含坏的三元组。

思路：可以证明大于四元组的点一定不满足条件，因为一定能够形成一个矩形，且让某个点在矩形中，此时对角两个点的曼哈顿距离就等于他们到中间点的曼哈顿距离和，所以只需对三元组和四元组进行判断即可。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int MAX = 2e5+50;
int a[MAX];

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n; scanf("%d", &n);
        for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
        int ans = n + n - 1;
        auto d = [&](int i, int j) -> int { return abs(a[i] - a[j]) + abs(i - j); };
        auto check = [&](int l, int r) -> int {
            for (int i = l; i <= r; i++) {
                for (int j = l; j <= r; j++) {
                    for (int k = l; k <= r; k++) {
                        if (i == j || i == k || j == k) continue;
                        if ((ll)d(i, j) == (ll)d(i, k) + (ll)d(k, j)) {
                            return 0;
                        }
                    }
                }
            }
            return 1;
        };
        for (int i = 3; i <= n; i++) {
            ans += check(i - 2, i);
            if (i >= 4) ans += check(i - 3, i);
        }
        printf("%d\n", ans);
    }
    return 0;
}
```

## D. Excellent Arrays

题意：

思路：

```cpp

```

## E. Stringforces

题意：

思路：

```cpp

```

## F. Jumping Around

题意：

思路：

```cpp

```


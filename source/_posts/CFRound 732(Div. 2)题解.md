---
title: CFRound 732(Div. 2)题解
date: 2021/07/12 23:42:21
categories: 
- 题解
tags: 
- 题解
- Codeforces
---

# Codeforces Round #732 (Div. 2)题解

传送门：[Dashboard - Codeforces Round #732 (Div. 2) - Codeforces](https://codeforces.com/contest/1546) 

<!--more-->

## A. AquaMoon and Two Arrays

题意：有两个数组$a, b$，可以做如下操作任意次：选择任意两个下标$i, j$，使$a[i] = a[i] - 1, a[j] = a[j] + 1$，判断是否能将$a$变成$b$。

思路：先求分别两个数组的和，如果和不相同，那么肯定不能成功。数据量比较小，可以直接进行模拟。

```cpp
#include <bits/stdc++.h>
using namespace std;
 
const int MAX = 105;
struct Node {
    int a, b, id;
    bool operator < (const Node& p) const {
        return a - b > p.a - p.b;
    }
}a[MAX];
int n;
 
int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        int suma = 0, sumb = 0;
        for (int i = 1; i <= n; i++) scanf("%d", &a[i].a), suma += a[i].a, a[i].id = i;
        for (int i = 1; i <= n; i++) scanf("%d", &a[i].b), sumb += a[i].b;
        if (suma != sumb) {
            puts("-1");
            continue;
        }
        sort(a + 1, a + n + 1);
        vector<pair<int, int>> ans;
        for (int i = 1; i <= n; i++) {
            for (int j = n; j >= 1; j--) {
                while (a[i].a > a[i].b && a[j].a < a[j].b) {
                    ans.emplace_back(a[i].id, a[j].id);
                    --a[i].a; ++a[j].a;
                }
            }
        }
        printf("%d\n", ans.size());
        for (auto& [a, b]: ans) printf("%d %d\n", a, b);
    }
    return 0;
}
```

## B. AquaMoon and Stolen String

题意：有$n$个字符串，其中一个字符串没找到与它匹配的字符串，接着有$n-1$个被打乱的能够匹配的字符串，需要找出那$n$个字符串里哪一个没被匹配。

思路：虚假的交互题。在$n+(n-1)$个串中的每一位一定会有两个字母匹配，那么剩下的那个字母一定是被偷的那个，所以直接对每一位字符进行统计即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
 
const int MAX = 1e5+50;
string s[MAX], ss[MAX];
int n, m;
 
int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= n; i++) cin >> s[i];
        for (int i = 1; i < n; i++) cin >> ss[i];
        string ans;
        for (int i = 0; i < m; i++) {
            vector<int> cnt(26, 0);
            for (int j = 1; j <= n; j++) {
                ++cnt[s[j][i] - 'a'];
            }
            for (int j = 1; j < n; j++) {
                --cnt[ss[j][i] - 'a'];
            }
            for (int i = 0; i < 26; i++) {
                if (cnt[i]) {
                    ans.push_back('a' + i);
                    break;
                }
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```

## C. AquaMoon and Strange Sort

题意：有$n$个数字，每个数字开始都朝向右边，可以进行以下操作任意次：交换两个相邻的数字，并改变它们的朝向。最后是否能使该序列非递减且都朝向右边。

思路：思维题。对于一个数字来讲，他要移动偶数次才能使最后的方向为右，我们可以统计所有数字开始的位置奇偶性，以及排过序的奇偶性，换一句话就是说所有原来在偶数位置的值最后也得在偶数位置，所有在奇数位置的值最后也得再奇数位置，才能够达到目标。

```cpp
#include <bits/stdc++.h>
using namespace std;
 
const int MAX = 1e5+50;
int n, a[MAX], b[MAX];
int odd1[MAX], odd2[MAX];
 
int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        memset(odd1, 0, sizeof(odd1));
        memset(odd2, 0, sizeof(odd2));
        for (int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
            b[i] = a[i];
            if (i & 1) ++odd2[a[i]];
        }
        sort(b + 1, b + n + 1);
        for (int i = 1; i <= n; i++) {
            if (i & 1) ++odd1[b[i]];
        }
        bool flag = true;
        for (int i = 1; i <= 100000; i++) {
            if (odd1[i] != odd2[i]) flag = false;
        }
        if (flag) puts("YES");
        else puts("NO");
    }
    return 0;
}
```

## D. AquaMoon and Chess

题意：有一个$01$序列，$1$可以跨过$1$个$1$到$0$的位置，求能得到多少种排列。

思路：可以把两个相邻的$1$看成是$1$组，不管怎么移动总的组数是不会变得，因为想要移动至少要有两个$1$。先预处理出组数，例如$11001110$→$(11)(0)(0)(11)(1)(0)$，由于$(1)$不会影响答案，所以我们只需要看$(11)$和$(0)$的个数，对两者进行排列组合，最后的答案就是$C_{n+m}^{m}$。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const ll MOD = 998244353;
const int MAX = 1e5+50;
int a[MAX], n;
ll fac[MAX], inv[MAX];

ll ksm(ll a, ll b) {
    ll res = 1;
    while (b) {
        if (b & 1LL) res = res * a % MOD;
        a = a * a % MOD;
        b >>= 1LL;
    }
    return res % MOD;
}

void init() {
    fac[0] = inv[1] = 1;
    for (int i = 1; i < MAX; i++) fac[i] = fac[i - 1] * i % MOD;
    inv[MAX - 1] = ksm(fac[MAX - 1], MOD - 2);
    for (int i = MAX - 2; i >= 0; i--) inv[i] = inv[i + 1] * (i + 1) % MOD;
}
 
ll C(int n, int m) {
    if (n == m || m == 0) return 1;
    return fac[n] * inv[m] % MOD * inv[n - m] % MOD;
}


int main() {
    init();
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) scanf("%1d", &a[i]);
        int cnt = 0, zero = count_if(a + 1, a + n + 1, [&](const int& x) { return !x; });
        for (int i = 2; i <= n; i++) {
            if (a[i - 1] == 1 && a[i] == 1) {
                ++cnt; ++i;
            }
        }
        printf("%lld\n", C(cnt + zero, cnt));
    }
    return 0;
}
```

## E. AquaMoon and Permutations

题意：

思路：

```cpp

```

## F. AquaMoon and Wrong Coordinate

题意：

思路：

```cpp

```


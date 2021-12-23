---
title: CFRound 734(Div. 3)题解
date: 2021/07/28 09:12:00
categories: 
- 题解
tags: 
- 题解
- Codeforces
---

# Codeforces Round #734 (Div. 3)题解

传送门：[Codeforces Round #734 (Div. 3)](https://codeforces.com/contest/1551) 

<!--more-->

## A. Polycarp and Coins

题意：寻找两个数$c_1,c_2$，使得$c_1 + 2  c_2 = n$，且$\mid c_1 - c_2 \mid$最小。

思路：很容易证明两数相差最多为$1$，我们可以先让$c_1 = c_2 = \lfloor \frac{n}{3} \rfloor$，再进行小范围调整。

```cpp
#include <bits/stdc++.h>
using namespace std;
 
int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n; scanf("%d", &n);
        int a = n / 3 + (n % 3 != 0);
        int b = n / 3;
        if (a + 2 * b != n) swap(a, b);
        printf("%d %d\n", a, b);
    }
    return 0;
}
```

## B1. Wonderful Coloring - 1

题意：给字母涂色，要满足相同字母的颜色不同，且不同颜色的个数相同。求最多能涂多少组颜色。一组颜色被认定为一红一绿。

思路：对字母个数分类讨论。对于个数$\ge 2$的字母，一定能凑出一组；对于其他字母，将它们个数相加，设和为$sum$，它们一共能凑出$\lfloor \frac{sum}{2} \rfloor$组，答案即两者的和。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 55;
char s[MAX];
int n;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%s", s + 1);
        n = strlen(s + 1);
        vector<int> cnt(26);
        for (int i = 1; i <= n; i++) ++cnt[s[i] - 'a'];
        int sum = 0, res = 0;
        for (int i = 0; i < 26; i++) {
            if (cnt[i] >= 2) ++res;
            else sum += cnt[i];
        }
        printf("%d\n", res + sum / 2);
    }
    return 0;
}
```

## B2. Wonderful Coloring - 2

题意：给数字涂色，与上题不同的是范围更大，且要输出方案。

思路：同样是分类讨论。对于数字个数$\ge k$的，直接将它们染上$1...k$，对于其他的数字，先将他们的位置储存下来，然后每$k$个进行染色，不足$k$个的不需要染色。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n, k; scanf("%d%d", &n, &k);
        vector<int> a(n);
        vector<int> col(n);
        vector<vector<int>> pos(n);
        map<int, int> mp;
        for (int i = 0; i < n; i++) {
            scanf("%d", &a[i]);
            --a[i];
            ++mp[a[i]];
            pos[a[i]].emplace_back(i);
        }
        vector<int> tmp;
        for (auto& [key, value]: mp) {
            if (value >= k) {
                for (int i = 0; i < k; i++) col[pos[key][i]] = i + 1;
            } else {
                for (auto& v: pos[key]) tmp.emplace_back(v);
            }
        }
        for (int i = 0; i < (int)tmp.size(); i++) {
            if ((i + 1) % k == 0) {
                for (int j = i - k + 1, c = 1; j <= i; j++, c++) {
                    col[tmp[j]] = c;
                }
            }
        }
        for (int i = 0; i < n; i++) printf("%d%c", col[i], " \n"[i == n - 1]);
    }
    return 0;
}
```

## C. Interesting Story

题意：从$n$个单词中取最多的单词，使得其中一个字母出现的次数严格大于其他字母出现的次数。

思路：因为只有五字母，可以枚举某一字母为出现最多的字母时，能够选择单词的最大数量。对于任一字母，我们先对单词进行排序，排序规则为当前字母与其他字母数量的差从大到小排序，然后按顺序选取单词，直到两者之差$\le 0$时退出，此时即当前字母能取到的最多单词数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e5+50;
int cnt[MAX][6], id[MAX], n;
char s[MAX];

int solve(int x) {
    sort(id + 1, id + n + 1, [&](int i, int j) {
        return 2 * cnt[i][x] - cnt[i][5] > 2 * cnt[j][x] - cnt[j][5];
    });
    int sum = 0, res = 0;
    for (int i = 1; i <= n; i++) {
        sum += 2 * cnt[id[i]][x] - cnt[id[i]][5];
        if (sum <= 0) break;
        ++res;
    }
    return res;
}

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%s", s + 1);
            memset(cnt[i], 0, sizeof(cnt[i]));
            int len = strlen(s + 1);
            for (int j = 1; j <= len; j++) ++cnt[i][s[j] - 'a'];
            cnt[i][5] = len;
        }
        iota(id, id + n + 5, 0);
        int res = 0;
        for (int i = 0; i < 5; i++) res = max(res, solve(i));
        printf("%d\n", res);
    }
    return 0;
}
```

## D1. Domino (easy version)

题意：

思路：

```cpp

```

## D2. Domino (hard version)

题意：

思路：

```cpp

```

## E. Fixed Points

题意：有一个长为$n$的序列$a$，可以对其进行操作，每次操作可以删除一个数字，并将该数字后面的所有数字往前移动一个位置。最少要移动多少位置能满足$a_i = i$的数量$\ge k$。

思路：很容易想到动态规划，$dp[i][j]$表示前$i$个数字删除$j$个的最大匹配数$(j \le i)$。考虑保留$a[i]$，如果前$i-1$个数字已经删除了$j$个那么$a[i]$会与$i-j$进行匹配；否则不保留$a[i]$，那么$dp[i][j] = dp[i - 1][j - 1]$，两者取最大值。有$dp$转移方程$dp[i][j] = max(dp[i - 1][j] + (a[i] == i - j), dp[i - 1][j - 1])$。最后的答案就是去寻找$dp[n][i] \ge k$的最小的$i$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e3+50;
int a[MAX], dp[MAX][MAX], n, k;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d%d", &n, &k);
        for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
        memset(dp, 0, sizeof(dp));
        for (int i = 1; i <= n; i++) {
            dp[i][0] = dp[i - 1][0] + (a[i] == i);
            for (int j = 1; j < i; j++) {
                dp[i][j] = max(dp[i - 1][j] + (a[i] == i - j), dp[i - 1][j - 1]);
            }
        }
        int res = -1;
        for (int i = 0; i <= n; i++) {
            if (dp[n][i] >= k) {
                res = i;
                break;
            }
        }
        printf("%d\n", res);
    }
    return 0;
}
```

## F. Equidistant Vertices

题意：

思路：

```cpp

```


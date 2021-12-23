---
title: CFRound 733(Div. 2)题解
date: 2021/07/19 09:02:00
categories: 
- 题解
tags: 
- 题解
- Codeforces
---

# Codeforces Round #733 (Div. 2)题解

传送门：[Codeforces Round #733 (Div. 1 + Div. 2, based on VK Cup 2021 - Elimination (Engine))](https://codeforces.com/contest/1530) 

<!--more-->

## A. Binary Decimal

题意：用最少的只包含$0, 1$的数字使得它们的和为$x$。求最少要多少个。

思路：只需要去找$x$中最大的那个数位即为答案。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n; scanf("%d", &n);
        int mx = 0;
        while (n) {
            mx = max(mx, n % 10);
            n /= 10;
        }
        printf("%d\n", mx);
    }
    return 0;
}
```

## B. Putting Plates

题意：有一个矩阵，只能在周围一圈放$1$，并且任意一个$1$的周围八个方向不能为$1$，求最多能放多少个$1$。

思路：直接进行模拟。先将第一行和最后一行放好$1$，第一列和最后一列只需要去贪心的放即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

int f[25][25];
int fx[8][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0}, {-1, -1}, {-1, 1}, {1, -1}, {1, 1} };

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        int n, m; scanf("%d%d", &n, &m);
        memset(f, 0, sizeof(f));
        for (int i = 1; i <= m; i += 2) f[1][i] = f[n][i] = 1;
        for (int i = 1; i <= n; i++) {
            if (f[i][1]) continue;
            bool flag = true;
            for (int j = 0; j < 8; j++) {
                int nx = i + fx[j][0];
                int ny = 1 + fx[j][1];
                if (f[nx][ny]) flag = false;
            }
            if (flag) f[i][1] = 1;
        }
        for (int i = 1; i <= n; i++) {
            if (f[i][m]) continue;
            bool flag = true;
            for (int j = 0; j < 8; j++) {
                int nx = i + fx[j][0];
                int ny = m + fx[j][1];
                if (f[nx][ny]) flag = false;
            }
            if (flag) f[i][m] = 1;
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                printf("%d", f[i][j]);
            }
            puts("");
        }
    }
    return 0;
}
```

## C. Pursuit

题意：每个人的得分定义为k个回合内，前$k - \lfloor \frac{k}{4} \rfloor$大的数的总和，求最少还要进行多少轮，我的得分才能不低于对手。

思路：对于多出来的那些轮，我必定是给自己$100$分，给对面$0$分，这样才能最小化回合数。因为分数的范围比较小，那么可以直接进行暴力，即枚举后面每轮的情况，直到我的分数不低于对面的分数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1e5+50;
int a[MAX], b[MAX], n;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
        for (int i = 1; i <= n; i++) scanf("%d", &b[i]);
        sort(a + 1, a + n + 1, greater<int>());
        sort(b + 1, b + n + 1, greater<int>());
        int sum1 = 0, sum2 = 0;
        for (int i = 1; i <= n - n / 4; i++) sum1 += a[i], sum2 += b[i];
        int cnt = 0, p1 = n - n / 4, p2 = n - n / 4;
        int tmp = n;
        while (sum1 < sum2) {
            ++cnt;
            ++n;
            sum1 += 100;
            if (n % 4 == 0) {
                if (p1 >= 1) sum1 -= a[p1];
                --p1;
            } else {
                ++p2;
                if (p2 <= tmp) sum2 += b[p2];
            }
        }
        printf("%d\n", cnt);
    }
    return 0;
}
```

## D. Secret Santa

题意：相当于给了一个序列$a$，需要找到一个排列$b$，使得$a_i = b_i$的数最多，且$i \ne b_i$。

思路：考虑贪心放，如果遇到还没放过的就放入$b$，但是这可能会让$i=b_i$。那么我们将不合法的数倒过来放，这样能保证如果剩余的数为奇数个，最多只会有$1$个$i = b_i$。接下来进行分类讨论，如果不存在$i=b_i$，那么直接去统计答案，能保证这样一定是最多的；否则，假设$p = b_p$ 为我要去找$i \ne b_i (i \ne p)$的位置，如果有这样的位置存在，我直接让$p$和它进行交换，这样不会对答案有影响；如果不存在，那么我要去找$a_i = a_p (i \ne p)$，这样子交换也不会对答案产生影响；如果前两种都找不到，那么我就随意交换，最后的答案会$-1$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e5+50;
int a[MAX], b[MAX], n;
bitset<MAX> v;

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        v.reset();
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) scanf("%d", &a[i]), b[i] = 0;
        if (n == 1) {
            puts("0");
            puts("1");
            continue;
        }
        for (int i = 1; i <= n; i++) {
            if (v[a[i]] || i == a[i]) continue;
            v[a[i]] = 1;
            b[i] = a[i];
        }
        for (int i = 1, p = n; i <= n; i++) {
            if (b[i]) continue;
            while (v[p]) --p;
            b[i] = p;
            --p;
        }
        int p = -1;
        for (int i = 1; i <= n; i++) {
            if (i == b[i]) {
                p = i;
                break;
            }
        }
        if (p != -1) {
            vector<int> pos;
            for (int i = 1; i <= n; i++) {
                if (a[i] != b[i] || i == p) pos.emplace_back(i);
                if (pos.size() >= 2) break;
            }
            if (pos.size() == 1) {
                bool flag = false;
                for (int j = 1; j <= n; j++) {
                    if (p == j) continue;
                    if (a[j] == a[p]) {
                        flag = true;
                        swap(b[j], b[p]);
                        break;
                    }
                }
                if (!flag) {
                    int q = (p == 1 ? 2 : 1);
                    swap(b[p], b[q]);
                }
            } else {
                int q = (p == pos[0] ? pos[1] : pos[0]);
                swap(b[p], b[q]);
            }
        }
        int cnt = 0;
        for (int i = 1; i <= n; i++) cnt += (a[i] == b[i]);
        printf("%d\n", cnt);
        for (int i = 1; i <= n; i++) printf("%d%c", b[i], " \n"[i == n]);
    }
    return 0;
}
```

## E. Minimax

题意：重新排列一个字符串使得该字符串$kmp$数组的最大值最小，且该字符串字典序最小。

思路：分类讨论。

一. 如果$s$中有出现仅一次的字符，那么将最小的那个出现仅一次的字符放到第一位，后面按照字典序放其他他字符。

二. 未出现以上情况，但最小的字符出现次数不超过$\lfloor \frac {\mid s \mid}{2} \rfloor + 1$，我们可以先在开头放两个该字符，后面不断按字典序放其他字符$+$该字符，例如$aaaabbcdd$，结果为$aababacdd$。

三. 未出现以上情况，且不同的的字符数量不超过$2$，例如$aaaabbbbbb$，结果为$abbbbbbaaa$。

四. 未出现以上情况，例如$aaaaaaaabbbccd$，结果为$abaaaaaaacbbcd$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1e5+50;
char s[MAX];
int n, cnt[26];

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%s", s + 1);
        n = strlen(s + 1);
        memset(cnt, 0, sizeof(cnt));
        for (int i = 1; i <= n; i++) ++cnt[s[i] - 'a'];
        int p = -1;
        for (int i = 0; i < 26; i++) {
            if (cnt[i] == 1) {
                p = i;
                break;
            }
        }
        if (p != -1) {
            --cnt[p];
            printf("%c", 'a' + p);
            for (int i = 0; i < 26; i++) {
                while (cnt[i]) --cnt[i], printf("%c", 'a' + i);
            }
            puts("");
            continue;
        }
        for (int i = 0; i < 26; i++) {
            if (cnt[i]) {
                p = i;
                break;
            }
        }
        if (cnt[p] <= n / 2 + 1) {
            printf("%c%c", 'a' + p, 'a' + p);
            cnt[p] -= 2;
            for (int i = 0; i < cnt[p]; i++) {
                for (int j = p + 1; j < 26; j++) {
                    if (cnt[j]) {
                        --cnt[j];
                        printf("%c%c", 'a' + j, 'a' + p);
                        break;
                    }
                }
            }
            for (int i = p + 1; i < 26; i++) {
                while (cnt[i]) --cnt[i], printf("%c", 'a' + i);
            }
            puts("");
            continue;
        }
        int sum = 0;
        for (int i = 0; i < 26; i++) sum += (cnt[i] > 0);
        if (sum <= 2) {
            --cnt[p];
            printf("%c", 'a' + p);
            for (int i = p + 1; i < 26; i++) {
                while (cnt[i]) --cnt[i], printf("%c", 'a' + i);
            }
            while (cnt[p]) --cnt[p], printf("%c", 'a' + p);
            puts("");
            continue;
        }
        --cnt[p];
        printf("%c", 'a' + p);
        int q = -1;
        for (int i = p + 1; i < 26; i++) {
            if (cnt[i]) {
                --cnt[i];
                q = i;
                printf("%c", 'a' + i);
                break;
            }
        }
        while (cnt[p]) --cnt[p], printf("%c", 'a' + p);
        for (int i = q + 1; i < 26; i++) {
            if (cnt[i]) {
                --cnt[i];
                printf("%c", 'a' + i);
                break;
            }
        }
        for (int i = 0; i < 26; i++) {
            while (cnt[i]) --cnt[i], printf("%c", 'a' + i);
        }
        puts("");
    }
    return 0;
}
```

## F. Bingo

题意：

思路：

```cpp

```

## G. What a Reversal

题意：

思路：

```cpp

```

## H. Turing's Award

题意：

思路：

```cpp

```


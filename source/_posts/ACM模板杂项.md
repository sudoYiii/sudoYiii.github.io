---
title: ACM模板杂项
date: 2021/07/07 15:40:57
categories: 
- 模板
tags: 
- 模板
- 杂项
---

<!--more-->

## 快读
```cpp
namespace fastIO {
  #define BUF_SIZE 100000
  //fread -> read
  bool IOerror = 0;
  inline char nc() {
    static char buf[BUF_SIZE], *p1 = buf + BUF_SIZE, *pend = buf + BUF_SIZE;
    if (p1 == pend) {
      p1 = buf;
      pend = buf + fread(buf, 1, BUF_SIZE, stdin);
      if (pend == p1) {
        IOerror = 1;
        return -1;
      }
    }
    return *p1++;
  }
  inline bool blank(char ch) {
    return ch == ' ' || ch == '\n' || ch == '\r' || ch == '\t';
  }
  template<class T> inline void read(T& x) {
    char ch;
    while (blank(ch = nc()));
    if (IOerror) return;
    for (x = ch - '0'; (ch = nc()) >= '0' && ch <= '9'; x = x * 10 + ch - '0');
  }
  #undef BUF_SIZE
};
using namespace fastIO;
//-----------------------------简易版-----------------------------------
inline __int128 read() {
  __int128 x = 0, f = 1; char ch = getchar();
  while (ch < '0' || ch > '9') { f = (ch == '-' ? -1 : 1); ch = getchar(); }
  while (ch >= '0' && ch <= '9') { x = x * 10 + ch - '0'; ch = getchar(); }
  return x * f;
}

void print(__int128 x) {
  if (x < 0) { putchar('-'); x = -x; }
  if (x > 9) print(x / 10);
  putchar(x % 10 + '0');
}

//关闭流同步,开启后不能和scanf、printf混用
ios::sync_with_stdio(false); cin.tie(nullptr);
```

## 前缀和 \ 差分(一维、二维)
```cpp
for (int i = 1; i <= n; i++) sum[i] = sum[i - 1] + a[i];
for (int i = 1; i <= n; i++)
  for (int j = 1; j <= m; j++)
    sum[i][j] = a[i][j] + sum[i-1][j] + sum[i][j - 1] - sum[i - 1][j - 1];

for (int i = 1; i <= n; i++) dif[i] = a[i] - a[i - 1];
for (int i = 1; i <= n; i++)
  for (int j = 1; j <= m; j++)
    dif[i][j] = a[i][j] - dif[i - 1][j] - dif[i][j - 1] + dif[i - 1][j - 1];
void modify(int aX, int aY, int bX, int bY, int x) {  //二维差分区间修改
  dif[aX][aY] += x;
  dif[aX][bY + 1] -= x;
  dif[bX + 1][aY] -= x;
  dif[bX + 1][bY + 1] += x;
}
```

## 手写lower_bound \ upper_bound
```cpp
int lowbou(int l, int r, int x) {
  while (l <= r) {
    int mid = (l + r) >> 1;
    if (a[mid] < x) l = mid + 1;
    else r = mid - 1;
  }
  return l;
}

int uppbou(int l, int r, int x) {
  while (l <= r) {
    int mid = (l + r) >> 1;
    if (a[mid] <= x) l = mid + 1;
    else r = mid - 1;
  }
  return l;
}
```

## 枚举大小为k的子集
```cpp
void subset(int n, int k) {
  int s = (1 << k) - 1;
  while (s < (1 << n)) {
    //...
    int x = s & (-s), y = s + x;
    s = (((s & (~y)) / x) >> 1) | y;
  }
}
```

## 整体二分
```cpp
//动态第k小问题
#include <bits/stdc++.h>
#define lowbit(x) ((x) & (-(x)))
using namespace std;

const int MAX = 1e5+50;
struct Q{
  int l, r, type, x, id;
  Q() {}
  Q(int _l, int _r, int _type, int _x, int _id = 0)
    : l(_l), r(_r), type(_type), x(_x), id(_id) {}
}q[MAX << 2], lq[MAX << 2], rq[MAX << 2];
int a[MAX], c[MAX], ans[MAX], n, m, cnt;

void add(int x, int y) {
  for (int i = x; i <= n; i += lowbit(i)) c[i] += y;
}

int query(int l, int r) {
  int res = 0;
  for (int i = r; i >= 1; i -= lowbit(i)) res += c[i];
  for (int i = l - 1; i >= 1; i -= lowbit(i)) res -= c[i];
  return res;
}

void solve(int l, int r, int L, int R) {
  if (l > r || L > R) return;
  if (l == r) {
    for (int i = L; i <= R; i++) if (q[i].type == 2) ans[q[i].id] = l;
    return;
  }
  int mid = (l + r) >> 1;
  int cnt1 = 0, cnt2 = 0;
  for (int i = L; i <= R; i++) {
    if (q[i].type != 2) {
      if (q[i].x <= mid) {
        add(q[i].l, q[i].type);
        lq[++cnt1] = q[i];
      } else rq[++cnt2] = q[i];
    } else {
      int num = query(q[i].l, q[i].r);
      if (q[i].x <= num) lq[++cnt1] = q[i];
      else rq[++cnt2] = q[i], rq[cnt2].x -= num;
    }
  }
  for (int i = 1; i <= cnt1; i++) if (lq[i].type != 2) add(lq[i].l, -lq[i].type);
  for (int i = L; i < L + cnt1; i++) q[i] = lq[i - L + 1];
  for (int i = L + cnt1; i <= R; i++) q[i] = rq[i - L - cnt1 + 1];
  solve(l, mid, L, L + cnt1 - 1);
  solve(mid + 1, r, L + cnt1, R);
}

int main(){
  scanf("%d%d", &n, &m);
  for (int i = 1; i <= n; i++) {
    scanf("%d", &a[i]);
    q[++cnt] = Q(i, i, 1, a[i]);
  }
  for (int i = 1; i <= m; i++) {
    char op; scanf(" %c", &op);
    if (op == 'C') {
      int x, y; scanf("%d%d", &x, &y);
      q[++cnt] = Q(x, x, -1, a[x]);
      q[++cnt] = Q(x, x, 1, y);
      a[x] = y;
    } else {
      int l, r, k; scanf("%d%d%d", &l, &r, &k);
      q[++cnt] = Q(l, r, 2, k, i);
    }
  }
  solve(-1e9, 1e9, 1, cnt);
  for (int i = 1; i <= m; i++) if (ans[i]) printf("%d\n", ans[i]);
  return 0;
}
```

#### 三分

```cpp
while (l <= r) {
  int lmid = l + (r - l) / 3;
  int rmid = r - (r - l) / 3;
  ll lans = f(lmid), rans = f(rmid);
  // 找凹函数极小值
  if(lans <= rans) r = rmid - 1;
  else l = lmid + 1;
  // 求凸函数的极大值
  if(lasn >= rans) l = lmid + 1;
  else r = rmid - 1;
  ans = min(ans, min(lans, rans));
}
```

## 莫队

```cpp
struct Q {
  int l, r, id, bel;
  bool operator < (const Q& a) const {
    if (bel != a.bel) return bel < a.bel;
    return (bel & 1) ? (r < a.r) : (r > a.r);
  }
}q[MAX];
int ans[MAX], n, m, block, res;

void add(int p) {}

void del(int p) {}

int main() {
  block = sqrt(n);
  for (int i = 1; i <= m; i++) {
    int l, r; scanf("%d%d", &l, &r);
    q[i].l = l; q[i].r = r;
    q[i].id = i; q[i].bel = q[i].l / block; 
  }
  int l = 1, r = 0;
  res = 0;
  for (int i = 1; i <= m; i++) {
    int ql = q[i].l, qr = q[i].r;
    while (l < ql) del(l++);
    while (l > ql) add(--l);
    while (r < qr) add(++r);
    while (r > qr) del(r--);
    ans[q[i].id] = res;
  }
  return 0;
}
```

## 模拟退火

```cpp
#include <bits/stdc++.h>
using namespace std;

const double eps = 1e-15;
const double down = 0.995;
const int MAX = 105;
struct Node {
  double x, y, w;
}node[MAX];
double resx, resy, resw;
int n;

double energy(double x, double y) {
  double res = 0;
  for (int i = 1; i <= n; i++) {
    double dx = x - node[i].x;
    double dy = y - node[i].y;
    res += sqrt(dx * dx + dy * dy);
  }
  return res;
}

void SAA() {
  double T = 3000;
  while (T > eps) {
    double ex = resx + (rand() * 2 - RAND_MAX) * T;
    double ey = resy + (rand() * 2 - RAND_MAX) * T;
    double ew = energy(ex, ey);
    double de = ew - resw;
    if (de < 0) {
      resx = ex;
      resy = ey;
      resw = ew;
    } else if (exp(-de / T) * RAND_MAX > rand()) {
      resx = ex;
      resy = ey;
    }
    T *= down;
  }
}

int main() {
  srand(time(0));
  int T; scanf("%d", &T);
  for (int _ = 1; _ <= T; _++) {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
      scanf("%lf%lf", &node[i].x, &node[i].y);
      resx += node[i].x;
      resy += node[i].y;
    }
    resx /= n;
    resy /= n;
    resw = energy(resx, resy);
    for (int i = 0; i < 10; i++) SAA();
    printf("%.0f\n", resw);
    if (_ != T) puts("");
  }
  return 0;
}
```

## __builtin系列函数

```cpp
__builtin_popcount(unsigned int x)  //返回x二进制1的个数
__builtin_parity(unsigned int x)  //返回x二进制1的个数的奇偶性
__builtin_ffs(unsigned int x)  //返回x二进制最后一个1的位置,从1开始
__builtin_ctz(unsigned int x)  //返回x二进制末尾0的个数
__builtin_clz(unsigned int x)  //返回前导0的个数
//使用long long可以在函数名后面加ll
```

## mod_int

```cpp
template<unsigned M_> struct ModInt {
  static constexpr unsigned M = M_;
  unsigned x;
  constexpr ModInt(): x(0U) {}
  constexpr ModInt(unsigned x_): x(x_ % M) {}
  constexpr ModInt(unsigned long long x_): x(x_ % M) {}
  constexpr ModInt(int x_): x(((x_ %= static_cast<int>(M)) < 0) ? (x_ + static_cast<int>(M)) : x_) {}
  constexpr ModInt(long long x_): x(((x_ %= static_cast<long long>(M)) < 0) ? (x_ + static_cast<long long>(M)) : x_) {}
  ModInt& operator+=(const ModInt& a) { x = ((x += a.x) >= M) ? (x - M) : x; return *this; }
  ModInt& operator-=(const ModInt& a) { x = ((x -= a.x) >= M) ? (x + M) : x; return *this; }
  ModInt& operator*=(const ModInt& a) { x = (static_cast<unsigned long long>(x) * a.x) % M; return *this; }
  ModInt& operator/=(const ModInt& a) { return (*this *= a.inv()); }
  ModInt pow(long long e) const {
    if (e < 0) return inv().pow(-e);
    ModInt a = *this, b = 1U; for (; e; e >>= 1) { if (e & 1) b *= a; a *= a; } return b;
  }
  ModInt inv() const {
    unsigned a = M, b = x; int y = 0, z = 1;
    while (b) {
      const unsigned q = a / b, c = a - q * b; a = b; b = c;
      const int w = y - static_cast<int>(q) * z; y = z; z = w;
    }
    assert(a == 1U); return ModInt(y);
  }
  ModInt operator+() const { return *this; }
  ModInt operator-() const { ModInt a; a.x = x ? (M - x) : 0U; return a; }
  ModInt operator+(const ModInt& a) const { return (ModInt(*this) += a); }
  ModInt operator-(const ModInt& a) const { return (ModInt(*this) -= a); }
  ModInt operator*(const ModInt& a) const { return (ModInt(*this) *= a); }
  ModInt operator/(const ModInt& a) const { return (ModInt(*this) /= a); }
  template<class T> friend ModInt operator+(T a, const ModInt& b) { return (ModInt(a) += b); }
  template<class T> friend ModInt operator-(T a, const ModInt& b) { return (ModInt(a) -= b); }
  template<class T> friend ModInt operator*(T a, const ModInt& b) { return (ModInt(a) *= b); }
  template<class T> friend ModInt operator/(T a, const ModInt& b) { return (ModInt(a) /= b); }
  explicit operator bool() const { return x; }
  bool operator==(const ModInt& a) const { return (x == a.x); }
  bool operator!=(const ModInt& a) const { return (x != a.x); }
  friend std::ostream& operator<<(std::ostream& os, const ModInt& a) { return os << a.x; }
};
using mint = ModInt<998244353>;
```

##### 奇奇怪怪的知识

$(a + b) = (a\ \&\ b) + (a\ |\ b)$；

$(a + b) = (a \oplus b) + 2 * (a\ \&\ b)$。


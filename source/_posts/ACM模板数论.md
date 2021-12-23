---
title: ACM模板数论
date: 2021/07/07 15:40:57
categories: 
- 模板
tags: 
- 模板
- 数论
---

<!--more-->

## 快速乘 && 快速幂

```cpp
ll mul(ll a, ll b, ll P) {
  return (a * b - (ll)((long double)a * b / P) * P + P) % P;
}

ll ksm(ll a, ll b, ll P) {
  ll res = 1;
  while (b) {
    if (b & 1LL) res = res * a % P;
    a = a * a % P;
    b >>= 1LL;
  }
  return res % P;
}
```

## 康托展开+康托逆展开

```cpp
int fac[MAX], a[MAX], b[MAX], n;
bool vis[MAX];

int contor() {
  int rank = 0;
  for (int i = 1; i <= n; i++) {
    int cnt = 0;
    for (int j = i + 1; j <= n; j++) cnt += a[i] > a[j];
    rank += cnt * fac[n - i];
  }
  return rank + 1;
}

void revContor(int rank) {
  --rank;
  memset(vis, false, sizeof(vis));
  for (int i = 1, tmp = rank, num; i <= n; i++) {
    num = tmp / fac[n - i];
    tmp = tmp % fac[n - i];
    for (int j = 1; j <= n; j++) {
      if (!vis[j] && !(num--)) {
        vis[j] = true;
        b[i] = j;
        break;
      }
    }
  }
}
```

## 欧拉筛+莫比乌斯函数

```cpp
int prime[MAX], cnt;
bool notp[MAX];  //true为合数,false为素数
int mu[MAX];

void init(int n) {
  notp[0] = notp[1] = true;
  //mu[1] = 1;
  for (int i = 2; i <= n; i++) {
    if (!notp[i]) { prime[cnt++] = i; /*mu[i] = -1;*/ }
    for (int j = 0; j < cnt && (ll)i * prime[j] <= (ll)n; j++) {
      notp[i * prime[j]] = true;
      if (i % prime[j] == 0) break;
      //else mu[i * prime[j]] = -mu[i];
    }
  }
}
```

## 欧拉函数

不超过$n$且与$n$互素的正整数的个数

欧拉定理：$gcd(a,m)=1$，则$a^{\varphi(m)} \equiv 1 \pmod{m}$。

扩展欧拉定理：$\displaystyle a^b \equiv \begin{cases} a^{b \bmod \varphi(p)}, & \gcd(a,\,p) = 1\newline a^b, & \gcd(a,\,p) \ne 1,\,b<\varphi(p) \newline a^{b \bmod \varphi(p) + \varphi(p)}, & \gcd(a,\,p) \ne1,\,b \ge \varphi(p) \end{cases} \pmod p$。

```cpp
int phi[MAX], prime[MAX], cnt;
bool notp[MAX];

void get_phi(int n) {
  phi[1] = 1;
  notp[0] = notp[1] = true;
  for (int i = 2; i <= n; i++) {
    if (!notp[i]) {
      prime[cnt++] = i;
      phi[i] = i - 1;
    }
    for (int j = 0; j < cnt && (ll)i * prime[j] <= (ll)n; j++) {
      notp[i * prime[j]] = true;
      if (i % prime[j] == 0) {
          phi[i * prime[j]] = phi[i] * prime[j];
          break;
      } else {
        phi[i * prime[j]] = phi[i] * (prime[j] - 1);
      }
    }
  }
}

//单个值欧拉函数
int euler_phi(int x) {
  int ans = x;
  for (int i = 2; i * i <= x; i++) {
    if (x % i == 0) {
      ans = ans / i * (i - 1);
      while (x % i == 0) x /= i;
    }
  }
  if (x > 1) ans = ans / x * (x - 1);
  return ans;
}
```

## 逆元预处理

```cpp
ll inv[MAX];

void init_inv() {
  inv[1] = 1;
  inv[MAX - 1] = ksm(fac[MAX - 1], MOD - 2);
  for (int i = MAX - 2; i >= 0; i--) inv[i] = inv[i + 1] * (i + 1) % MOD;
}
```

## exGCD

用来求解 $ax+by=\gcd(a,b)$ 方程的通解

定理1：$ax+by=c$ 与 $ax \equiv c\pmod b$ 等价，有整数解的充要条件为 $\gcd(a,b) \mid c$

定理2：若$\gcd(a,b)=1$，且 $x_0$、$y_0$ 为方程 $ax+by=c$ 的一组解，则该方程的任意解可表示为：$x=x_0+bt$，$y=y_0-at$

```cpp
void exgcd(int a, int b, int& x, int& y) {
  if (!b) { x = 1; y = 0; return; }
  exgcd(b, a % b, x, y);
  int t = x; x = y; y = t - a / b * y;
}
```

## CRT

求解线性方程组$\begin{cases} x &\equiv a_1 \pmod {n_1} \newline x &\equiv a_2 \pmod {n_2} \newline   &\vdots \newline x &\equiv a_k \pmod {n_k} \end{cases}$

```cpp
ll a[35], m[35];

ll CRT(int k, ll a[], ll m[]) {
  a[1] %= m[1];
  for (int i = 2; i <= k; i++) {
    a[i] %= m[i];
    ll d = __gcd(m[1], m[i]), x, y;
    exgcd(m[1], m[i], x, y);
    if ((a[i] - a[1]) % d != 0LL) return -1;
    ll t = m[i] / d;
    x = ((a[i] - a[1]) / d * x % t + t) % t;
    a[1] += x * m[1];
    m[1] *= m[i] / d;
    a[1] = (a[1] % m[1] + m[1]) % m[1];
  }
  return a[1];
}

int main() {
  int n; scanf("%d", &n);
  for (int i = 1; i <= n; i++) scanf("%lld%lld", &m[i], &a[i]);
  printf("%lld\n", CRT(n, a, m));
  return 0;
}
```

## 整除分块

$\sum_{i=1}^{n}\left \lfloor \frac{n}{i} \right \rfloor$ 

```cpp
for (ll l = 1, r; l <= n; l = r + 1) {
  r = n / (n / l);
  ans += n / l * (r - l + 1);
}
```

## 组合数预处理

```c++
ll fac[MAX], inv[MAX];

void init() {
  fac[0] = inv[1] = 1;
  for (int i = 1; i < MAX; i++) fac[i] = fac[i - 1] * i % MOD;
  inv[MAX - 1] = ksm(fac[MAX - 1], MOD - 2);
  for (int i = MAX - 2; i >= 0; i--) inv[i] = inv[i + 1] * (i + 1) % MOD;
}

ll nCr(int n, int m) {
  if (n == m || m == 0) return 1;
  return fac[n] * inv[m] % MOD * inv[n - m] % MOD;
}
```

## Lucas(大组合数)

```cpp
ll nCr(ll n, ll m, ll P) {
  if (m >= n) return 0;
  return fac[n] * inv[m] % P * inv[n - m] % P;
}

ll Lucas(ll n, ll m, ll P) {
  if (m == 0LL) return 1;
  return nCr(n % P, m % P, P) * Lucas(n / P, m / P, P) % P;
}
```

## BSGS

用来求 $a^x \equiv b \pmod p$ 最小的$x$，时间复杂度 $O(\sqrt p)$

```cpp
ll BSGS(ll a, ll b, ll m, ll k = 1) {  //额外带一个系数
  static map<ll, ll> hs;
  hs.clear();
  ll cur = 1, t = (ll)sqrt(m) + 1;
  for (int B = 1; B <= t; B++) {
    (cur *= a) %= m;
    hs[b * cur % m] = B;  //哈希表中存B的值
  }
  ll now = cur * k % m;
  for (int A = 1; A <= t; A++) {
    auto it = hs.find(now);
    if (it != hs.end()) return A * t - it->second;
    (now *= cur) %= m;
  }
  return LLONG_MIN;  //这里因为要多次加1, 要返回更小的负数
}

ll exBSGS(ll a, ll b, ll m, ll k = 1) {
  ll A = (a %= m), B = (b %= m), M = m;
  if (b == 1LL) return 0;
  ll cur = 1 % m;
  for (int i = 0;; i++) {
    if (cur == B) return i;
    cur = cur * A % M;
    ll d = __gcd(a, m);
    if (b % d) return LLONG_MIN;
    if (d == 1LL) return BSGS(a, b, m, k * a % m) + i + 1;
    k = k * a / d % m, b /= d, m /= d;  //相当于在递归求解exBSGS(a, b / d, m / d, k * a / d % m)
  }
}
```

## 杜教BM

用来求解线性递推式,放进去的前几项越多，答案越准

```cpp
namespace BM {
const int N = 1e3 + 5;
ll res[N], base[N], _c[N], _md[N];
vector<int> Md;

void mul(ll* a, ll* b, int k) {
  for (int i = 0; i < k + k; i++) _c[i] = 0;
  for (int i = 0; i < k; i++) if (a[i])
    for (int j = 0; j < k; j++)
      _c[i + j] = (_c[i + j] + a[i] * b[j]) % MOD;
  for (int i = k + k - 1; i >= k; i--) if (_c[i])
    for (int j = 0; j < Md.size(); j++)
      _c[i - k + Md[j]] = (_c[i - k + Md[j]] - _c[i] * _md[Md[j]]) % MOD;
  for (int i = 0; i < k; i++) a[i] = _c[i];
}

int solve(ll n, vector<int> a, vector<int> b) {
  ll ans = 0, pnt = 0;
  int k = a.size();
  for (int i = 0; i < k; i++) _md[k - 1 - i] = -a[i];
  _md[k] = 1;
  Md.clear();
  for (int i = 0; i < k; i++) if (_md[i] != 0) Md.push_back(i);
  for (int i = 0; i < k; i++) res[i] = base[i] = 0;
  res[0] = 1;
  while ((1ll << pnt) <= n) pnt++;
  for (int p = pnt; p >= 0; p--) {
    mul(res, res, k);
    if ((n >> p) & 1){
      for (int i = k - 1; i >= 0; i--) res[i + 1] = res[i];
      res[0] = 0;
      for (int j = 0; j < Md.size(); j++)
        res[Md[j]] = (res[Md[j]] - res[k] * _md[Md[j]]) % MOD;
    }
  }
  for (int i = 0; i < k; i++) ans = (ans + res[i] * b[i]) % MOD;
  if (ans < 0) ans += MOD;
  return ans;
}

vector<int> CalBM(vector<int> s) {
  vector<int> C(1, 1), B(1, 1);
  int L = 0, m = 1, b = 1;
  for (int n = 0; n < s.size(); n++) {
    ll d = 0;
    for (int i = 0; i <= L; i++) d = (d + 1ll * C[i] * s[n - i]) % MOD;
    if (d == 0) ++m;
    else if ((L << 1) <= n) {
      vector<int> T = C;
      ll c = MOD - d * ksm(b, MOD - 2) % MOD;
      while (C.size() < B.size() + m) C.push_back(0);
      for (int i = 0; i < B.size(); i++)
        C[i + m] = (C[i + m] + c * B[i]) % MOD;
      L = n + 1 - L; B = T; b = d; m = 1;
    } else {
      ll c = MOD - d * ksm(b, MOD - 2) % MOD;
      while (C.size() < B.size() + m) C.push_back(0);
      for (int i = 0; i < B.size(); i++)
        C[i + m] = (C[i + m] + c * B[i]) % MOD;
      ++m;
    }
  }
  return C;
}

int giao(vector<int> a, ll n) {
  vector<int> c = CalBM(a);
  c.erase(c.begin());
  for (int i = 0; i < c.size(); i++) c[i] = (MOD - c[i]) % MOD;
  return solve(n, c, vector<int>(a.begin(), a.begin() + c.size()));
}
}
```

## Miller-Robin(判断大质数)+Pollard-Rho

```cpp
ll max_factor, n;

ll ksm(ll a, ll b, ll P) {
  ll res = 1;
  while (b) {
    if (b & 1LL) res = (i128)res * a % P;
    a = (i128)a * a % P;
    b >>= 1LL;
  }
  return res % P;
}

bool Miller_Rabin(ll p) {
  if (p < 2LL) return false;
  if (p == 2LL || p == 3LL) return true;
  ll d = p - 1, r = 0;
  while (!(d & 1LL)) ++r, d >>= 1LL;
  for (ll k = 0; k < 10; ++k) {
    ll a = rand() % (p - 2) + 2;
    ll x = ksm(a, d, p);
    if (x == 1 || x == p - 1) continue;
    for (int i = 0; i < r - 1; ++i) {
      x = (i128)x * x % p;
      if (x == p - 1) break;
    }
    if (x != p - 1) return false;
  }
  return true;
}

ll f(ll x, ll c, ll n) { return ((i128)x * x + c) % n; }

ll Pollard_Rho(ll x) {
  ll s = 0, t = 0;
  ll c = (ll)rand() % (x - 1) + 1;
  int step = 0, goal = 1;
  ll val = 1;
  for (goal = 1;; goal <<= 1, s = t, val = 1) {
    for (step = 1; step <= goal; ++step) {
      t = f(t, c, x);
      val = (i128)val * abs(t - s) % x;
      if ((step % 127) == 0) {
        ll d = __gcd(val, x);
        if (d > 1) return d;
      }
    }
    ll d = __gcd(val, x);
    if (d > 1) return d;
  }
}

void fac(ll x) {
  if (x <= max_factor || x < 2) return;
  if (Miller_Rabin(x)) {
    max_factor = max(max_factor, x);
    return;
  }
  ll p = x;
  while (p >= x) p = Pollard_Rho(x);
  while ((x % p) == 0) x /= p;
  fac(x), fac(p);
}

int main() {
  srand(time(0));
  int T; scanf("%d", &T);
  while (T--) {
    scanf("%lld", &n);
    max_factor = 0;
    fac(n);
    if (max_factor == n) puts("Prime");
    else printf("%lld\n", max_factor);
  }
  return 0;
}
```

## 矩阵

```cpp
struct Mat {
  int row, col;
  ll mat[MAX][MAX];
  Mat(int _r, int _c): row(_r), col(_c) {
    memset(mat, 0, sizeof(mat));
  }
  void base() {
    memset(mat, 0, sizeof(mat));
    for (int i = 1; i <= row; i++) mat[i][i] = 1;
  }
};

Mat mul(Mat a, Mat b) {  //要保证a的列数==b的行数
  Mat res = Mat(a.row, b.col);
  for (int i = 1; i <= res.row; i++)
    for (int j = 1; j <= res.col; j++)
      for (int k = 1; k <= a.col; k++)
        res.mat[i][j] = (res.mat[i][j] + a.mat[i][k] * b.mat[k][j] % MOD) % MOD;
  return res;
}

Mat ksm(Mat a, ll b) {
  Mat res = Mat(a.row, a.col);
  res.base();
  while (b) {
    if (b & 1LL) res = mul(res, a);
    a = mul(a, a);
    b >>= 1LL;
  }
  return res;
}
```

## 高斯消元

```cpp
const double eps = 1e-12;
const int MAX = 55;
double a[MAX][MAX], x[MAX];  //a方程组,x解集
int n;

int sgn(double x) {  //判断一个数的正负
  if (fabs(x) < eps) return 0;  //x = 0
  if (x < 0) return -1;  //x < 0
  return 1;  //x > 0
}

int gauss(int n) {  //n个方程,n个变元;0唯一解,-1无解,>=1多解
  int nwline = 0;
  for (int k = 0; k < n; k++) {
    int maxi = nwline;
    for (int i = nwline + 1; i < n; i++) {
      if (fabs(a[i][k]) > fabs(a[maxi][k])) maxi = i;
    }
    if (sgn(a[maxi][k]) == 0) continue;
    for (int j = 0; j < n + 1; j++) {
      swap(a[nwline][j], a[maxi][j]);
    }
    for (int i = 0; i < n; i++) {
      if (i == nwline) continue;
      double mul = a[i][k] / a[nwline][k];
      for (int j = k + 1; j < n + 1; j++) {
        a[i][j] -= a[nwline][j] * mul;
      }
    }
    ++nwline;
  }
  for (int i = nwline; i < n; i++) {
    if (sgn(a[i][n]) != 0) return -1;
  }
  if (nwline < n) return n - nwline;
  for (int i = 0; i < n; i++) {
    x[i] = a[i][n] / a[i][i];
    if (sgn(x[i]) == 0) x[i] = 0;
  }
  return 0;
}
```

## 高斯异或方程组

```cpp
int a[MAX][MAX], x[MAX], n;
bool freeX[MAX];

int gauss(int n, int m) {
  for (int i = 0; i < m + 5; i++) x[i] = 0, freeX[i] = false;
  int row = 0, col = 0, num = 0;
  for (; row < n && col < m; row++, col++) {
    int maxRow = row;
    for (int i = row + 1; i < n; i++)
      if (abs(a[i][col]) > abs(a[maxRow][col])) maxRow = i;
    if (maxRow != row) {
      for (int i = row; i < m + 1; i++)
        swap(a[row][i], a[maxRow][i]);
    }
    if (a[row][col] == 0) {
      freeX[num++] = col;
      --row;
      continue;
    }
    for (int i = row + 1; i < n; i++)
      if (a[i][col] != 0)
        for (int j = col; j < m + 1; j++) a[i][j] ^= a[row][j];
  }
  for (int i = row; i < n; i++) if (a[i][col] != 0) return -1;
  if (row < m) return m - row;
  for (int i = m - 1; i >= 0; i--) {
    x[i] = a[i][m];
    for (int j = i + 1; j < m; j++) x[i] ^= (a[i][j] && x[j]);
  }
  return 0;
}
```

## FFT

```cpp
const double PI = acos(-1.0);
const int MAX = 5e5 + 50;
struct Complex{
  double x, y;
  Complex(double _x = 0, double _y = 0): x(_x), y(_y) {}
  //复数乘法:模长相乘,幅度相加
  Complex operator * (Complex a) { return Complex(x * a.x - y * a.y, x * a.y + y * a.x); }
  Complex operator + (Complex a) { return Complex(x + a.x, y + a.y); }
  Complex operator - (Complex a) { return Complex(x - a.x, y - a.y); }
}a[MAX], b[MAX];
int n, m, limit = 1;
int R[MAX], L;  //二进制翻转及其位数

void FFT(Complex A[], int type) {  //type=1表示从系数变为点值; type=-1表示从点值变为系数
  for (int i = 0; i < limit; i++) if (i < R[i]) swap(A[i], A[R[i]]);
  for (int mid = 1; mid < limit; mid <<= 1) {
    Complex Wn(cos(PI / mid), type * sin(PI / mid));
    for (int len = mid << 1, pos = 0; pos < limit; pos += len) {
      Complex w(1, 0);
      for (int k = 0; k < mid; k++, w = w * Wn) {
        Complex x = A[pos + k];
        Complex y = w * A[pos + mid + k];
        A[pos + k] = x + y;
        A[pos + mid + k] = x - y;
      }
    }
  }
  if (type == 1) return;
  for (int i = 0; i <= limit; i++) a[i].x /= limit;
}

void Poly_mul(Complex a[], Complex b[], int deg) {
  while (limit <= deg) limit <<= 1, ++L;
  for (int i = 0; i < limit; i++) R[i] = (R[i >> 1] >> 1) | ((i & 1) << (L - 1));
  FFT(a, 1); FFT(b, 1);
  for (int i = 0; i <= limit; i++) a[i] = a[i] * b[i];
  FFT(a, -1);
}

int main(){
  scanf("%d%d", &n, &m);
  for (int i = 0; i <= n; i++) scanf("%lf", &a[i].x);
  for (int i = 0; i <= m; i++) scanf("%lf", &b[i].x);
  Poly_mul(a, b, n + m);
  for (int i = 0; i <= n+m; i++) printf("%d%c", (int)(a[i].x + 0.5), " \n"[i == n + m]);
  return 0;
}
```

## NTT

```cpp
const ll MOD = 998244353, G = 3, Gi = 332748118;  //Gi是G的除法逆元
const int MAX = 4e6 + 50;
int n, m, limit = 1, L, R[MAX];
ll a[MAX], b[MAX];

void NTT(ll A[], int type) {  //type=1表示从系数变为点值; type=-1表示从点值变为系数
  for (int i = 0; i < limit; i++) if (i < R[i]) swap(A[i], A[R[i]]);
  for (int mid = 1; mid < limit; mid <<= 1) {
    ll Wn = ksm(G, (MOD - 1) / (mid * 2));
    if (type == -1) Wn = inv(Wn);
    for (int len = mid << 1, pos = 0; pos < limit; pos += len) {
      ll w = 1;
      for (int k = 0; k < mid; k++, w = w * Wn % MOD) {
        ll x = A[pos + k], y = w * A[pos + mid + k] % MOD;
        A[pos + k] = (x + y) % MOD;
        A[pos + k + mid] = (x - y + MOD) % MOD;
      }
    }
  }
  if (type == 1) return;
  ll limit_inv = inv(limit);
  for (int i = 0; i < limit; i++) A[i] = A[i] * limit_inv % MOD;
}

void Poly_mul(ll a[], ll b[], int deg) {
  while (limit <= deg) limit <<= 1, ++L;
  for (int i = 0; i < limit; i++) R[i] = (R[i >> 1] >> 1) | ((i & 1) << (L - 1));
  NTT(a, 1); NTT(b, 1);
  for (int i = 0; i < limit; i++) a[i] = a[i] * b[i] % MOD;
  NTT(a, -1);
}

int main() {
  scanf("%d%d", &n, &m);
  for (int i = 0; i <= n; i++) scanf("%lld", &a[i]), a[i] = (a[i] + MOD) % MOD;
  for (int i = 0; i <= m; i++) scanf("%lld", &b[i]), b[i] = (b[i] + MOD) % MOD;
  Poly_mul(a, b, n + m);
  for (int i = 0; i <= n + m; i++) printf("%lld%c", a[i], " \n"[i == n + m]);
  return 0;
}
```

## FWT

```cpp
const ll MOD = 998244353, inv2 = 499122177;
const int MAX = (1 << 17) + 50;
ll a[MAX], b[MAX], c[MAX];
int n;

void FWT_OR(ll a[], int len, int opt) {
  for (int i = 1; i < len; i <<= 1) {
    for (int j = 0, R = i << 1; j < len; j += R) {
      for (int k = 0; k < i; k++) {
        if (opt == 1) a[i + j + k] = (a[i + j + k] + a[j + k]) % MOD;
        else a[i + j + k] = (a[i + j + k] - a[j + k] + MOD) % MOD;
      }
    }
  }
}

void FWT_AND(ll a[], int len, int opt) {
  for (int i = 1; i < len; i <<= 1) {
    for (int j = 0, R = i << 1; j < len; j += R) {
      for (int k = 0; k < i; k++) {
        if (opt == 1) a[j + k] = (a[j + k] + a[i + j + k]) % MOD;
        else a[j + k] = (a[j + k] - a[i + j + k] + MOD) % MOD;
      }
    }
  }
}

void FWT_XOR(ll a[], int len, int opt) {
  for (int i = 1; i < len; i <<= 1) {
    for (int j = 0, R = i << 1; j < len; j += R) {
      for (int k = 0; k < i; k++) {
        ll x = a[j + k], y = a[i + j + k];
        if (opt == 1) {
          a[j + k] = (x + y) % MOD;
          a[i + j + k] = (x - y + MOD) % MOD;
        } else {
          a[j + k] = (x + y) % MOD * inv2 % MOD;
          a[i + j + k] = (x - y + MOD) % MOD * inv2 % MOD;
        }
      }
    }
  }
}
// OR, AND, XOR操作同理, len不足2的幂次用0补齐
int main() {
  scanf("%d", &n);
  int len = 1 << n;
  for (int i = 0; i < len; i++) scanf("%lld", &a[i]);
  for (int i = 0; i < len; i++) scanf("%lld", &b[i]);
  FWT_OR(a, len, 1); FWT_OR(b, len, 1);
  for (int i = 0; i < len; i++) c[i] = a[i] * b[i] % MOD;
  FWT_OR(c, len, -1);
  for (int i = 0; i < len; i++) printf("%lld%c", c[i], " \n"[i == len - 1]);
  FWT_OR(a, len, -1); FWT_OR(b, len, -1);
  return 0;
}
```

## 线性基

```cpp
template<class T> struct LinearBasis {
  static const int LEN = 32;
  T p[LEN];

  LinearBasis() { memset(p, 0, sizeof(p)); }

  void insert(T x) {
    for (int i = LEN - 1; i >= 0; i--) {
      if (!((x >> i) & 1)) continue;
      if (!p[i]) { p[i] = x; return; }
      x ^= p[i];
    }
  }

  void mergeFrom(const LinearBasis& oth) {
    for (int i = LEN - 1; i >= 0; i--) insert(oth.p[i]);
  }

  T queryMax() {
    T res = 0;
    for (int i = LEN - 1; i >= 0; i--) res = max(res, res ^ p[i]);
    return res;
  }
};
```


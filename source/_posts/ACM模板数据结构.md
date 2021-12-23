---
title: ACM模板数据结构
date: 2021/07/07 15:42:57
categories: 
- 模板
tags: 
- 模板
- 数据结构
---

<!--more-->

## 并查集

```cpp
struct UF {
  vector<int> fa, sz;
  int n, comp_cnt;

  UF(int _n): n(_n), comp_cnt(_n), fa(_n), sz(_n, 1) {
    iota(fa.begin(), fa.end(), 0);
  }

  int find(int x) {
    return fa[x] == x ? x : (fa[x] = find(fa[x]));
  }

  bool merge(int x, int y) {
    x = find(x);
    y = find(y);
    if (x == y) {
      return false;
    }
    if (sz[x] < sz[y]) {
      swap(x, y);
    }
    fa[y] = x;
    sz[x] += sz[y];
    --comp_cnt;
    return true;
  }

  bool same(int x, int y) {
    return find(x) == find(y);
  }
};
```

## 一维RMQ

```cpp
int ST[MAX][25], a[MAX], logn[MAX], n, m;

void init_ST() {
  logn[1] = 0; logn[2] = 1;
  for (int i = 3; i <= n; i++) logn[i] = logn[i / 2] + 1;
  for (int i = 1; i <= n; i++) ST[i][0] = a[i];
  for (int j = 1; j <= logn[n]; j++) {
    for (int i = 1; i + (1 << j) - 1 <= n; i++) {
      ST[i][j] = max(ST[i][j - 1], ST[i + (1 << (j - 1))][j - 1]);
    }
  }
}

int RMQ(int l, int r) {
  int s = logn[r - l + 1];
  return max(ST[l][s], ST[r - (1 << s) + 1][s]);
}
```

## 二维RMQ

```cpp
int ST[MAX][MAX][9][9], a[MAX][MAX], n, m;
int logn[MAX];

int calc(int x, int y, int xx, int yy, int p, int q) {
  return max(
    max(ST[x][y][p][q], ST[xx - (1 << p) + 1][yy - (1 << q) + 1][p][q]),
    max(ST[xx - (1 << p) + 1][y][p][q], ST[x][yy - (1 << q) + 1][p][q])
  );
}

void init_ST() {
  logn[1] = 0; logn[2] = 1;
  for (int i = 3; i < MAX; i++) logn[i] = logn[i >> 1] + 1;
  for (int x = 0; x <= logn[n]; x++)
  for (int y = 0; y <= logn[m]; y++)
  for (int i = 1; i + (1 << x) - 1 <= n; i++) {
    for (int j = 1; j + (1 << y) - 1 <= m; j++) {
      if (!x && !y) { ST[i][j][x][y] = a[i][j]; continue; }
      ST[i][j][x][y] = calc(i, j, i + (1 << x) - 1, j + (1 << y) - 1, max(x - 1, 0), max(y - 1, 0));
    }
  }
}

int RMQ(int x, int y, int xx, int yy) {
  return calc(x, y, xx, yy, logn[xx - x + 1], logn[yy - y + 1]);
}
```

## 分块

```cpp
int n, m, tot, bel[MAX], L[MAX], R[MAX];
ll a[MAX], sum[MAX], c[MAX];

void init(){
  int persz = (int)sqrt(n);
  for (int i = 1; i <= n; i += persz){
    ++tot; L[tot] = i; R[tot] = min(n, i + persz - 1);
    for (int j = L[tot]; j <= R[tot]; j++){
      bel[j] = tot;
      sum[tot] += a[j];
    }
  }
}

void add(int l, int r, ll x){
  if (bel[l] == bel[r]) {
    for (int i = l; i <= r; i++) a[i] += x, sum[bel[i]] += x;
  } else {
    for (int i = bel[l] + 1; i <= bel[r] - 1; i++) sum[i] += x * (R[i] - L[i] + 1), c[i] += x;
    for (int i = l; i <= R[bel[l]]; i++) a[i] += x, sum[bel[i]] += x;
    for (int i = L[bel[r]]; i <= r; i++) a[i] += x, sum[bel[i]] += x;
  }
}

ll query(int l, int r){
  ll res = 0;
  if (bel[l] == bel[r]){
    for (int i = l; i <= r; i++) res += a[i] + c[bel[i]];
  }else{
    for (int i = bel[l] + 1; i <= bel[r] - 1; i++) res += sum[i];
    for (int i = l; i <= R[bel[l]]; i++) res += a[i] + c[bel[i]];
    for (int i = L[bel[r]]; i <= r; i++) res += a[i] + c[bel[i]];
  }
  return res;
}
```

## jls线段树

```cpp
#include <bits/stdc++.h>
#define ls k << 1
#define rs k << 1 | 1
#define lson k << 1, l, mid
#define rson k << 1 | 1, mid + 1, r
#define ll long long
using namespace std;

const int MAX = 1e6 + 50;
struct Node {
  int l, r, mx, se, mxcnt, tag;
  ll sum;
} tree[MAX << 2];
int a[MAX], n, m;

void pushup(int k) {
  tree[k].sum = tree[ls].sum + tree[rs].sum;
  if (tree[ls].mx == tree[rs].mx) {
    tree[k].mx = tree[ls].mx;
    tree[k].se = max(tree[ls].se, tree[rs].se);
    tree[k].mxcnt = tree[ls].mxcnt + tree[rs].mxcnt;
  } else if (tree[ls].mx > tree[rs].mx) {
    tree[k].mx = tree[ls].mx;
    tree[k].se = max(tree[ls].se, tree[rs].mx);
    tree[k].mxcnt = tree[ls].mxcnt;
  } else {
    tree[k].mx = tree[rs].mx;
    tree[k].se = max(tree[ls].mx, tree[rs].se);
    tree[k].mxcnt = tree[rs].mxcnt;
  }
}

void pushdown(int k) {
  if (tree[k].tag == -1) return;
  if (tree[ls].mx > tree[k].tag) {
    tree[ls].sum -= (ll)(tree[ls].mx - tree[k].tag) * tree[ls].mxcnt;
    tree[ls].mx = tree[ls].tag = tree[k].tag;
  }
  if (tree[rs].mx > tree[k].tag) {
    tree[rs].sum -= (ll)(tree[rs].mx - tree[k].tag) * tree[rs].mxcnt;
    tree[rs].mx = tree[rs].tag = tree[k].tag;
  }
  tree[k].tag = -1;
}

void build(int k, int l, int r) {
  tree[k].l = l; tree[k].r = r;
  tree[k].tag = -1;
  if (l == r) {
    tree[k].sum = tree[k].mx = a[l];
    tree[k].se = -1;
    tree[k].mxcnt = 1;
    return;
  }
  int mid = (l + r) >> 1;
  build(lson);
  build(rson);
  pushup(k);
}

void modify(int k, int l, int r, int x) {
  if (tree[k].mx <= x) return;
  if (l > tree[k].r || r < tree[k].l) return;
  if (l <= tree[k].l && tree[k].r <= r && tree[k].se < x) {
    if (tree[k].mx > x) {
      tree[k].sum -= (ll)(tree[k].mx - x) * tree[k].mxcnt;
      tree[k].mx = tree[k].tag = x;
    }
    return;
  }
  pushdown(k);
  int mid = (l + r) >> 1;
  modify(ls, l, r, x);
  modify(rs, l, r, x);
  pushup(k);
}

int querymx(int k, int l, int r) {
  if (tree[k].l == l && tree[k].r == r) return tree[k].mx;
  pushdown(k);
  int mid = (tree[k].l + tree[k].r) >> 1; 
  if (r <= mid) return querymx(ls, l, r);
  else if (l > mid) return querymx(rs, l, r);
  else return max(querymx(lson), querymx(rson));
}

ll querysum(int k, int l, int r) {
  if (tree[k].l == l && tree[k].r == r) return tree[k].sum;
  pushdown(k);
  int mid = (tree[k].l + tree[k].r) >> 1; 
  if (r <= mid) return querysum(ls, l, r);
  else if (l > mid) return querysum(rs, l, r);
  else return querysum(lson) + querysum(rson);
}

int main() {
  int T; scanf("%d", &T);
  while (T--) {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    build(1, 1, n);
    while (m--) {
      int op, l, r; scanf("%d%d%d", &op, &l, &r);
      if (op == 0) {
        int t; scanf("%d", &t);
        modify(1, l, r, t);
      } else if (op == 1) {
        printf("%d\n", querymx(1, l, r));
      } else {
        printf("%lld\n", querysum(1, l, r));
      }
    }
  }
  return 0;
}
```

## 主席树

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 2e5 + 10;
struct Node {
  int l, r, sum;
}tree[MAX << 5];
int root[MAX], idx;  //每个版本树的根
int a[MAX], b[MAX];

void Insert(int l, int r, int pre, int& now, int x) {
  now = ++idx;
  tree[now] = tree[pre];
  tree[now].sum++;
  if (l == r) return;
  int mid = (l + r) >> 1;
  if (x <= mid) Insert(l, mid, tree[pre].l, tree[now].l, x);
  else Insert(mid+1, r, tree[pre].r, tree[now].r, x);
}

int Query(int l, int r, int L, int R, int k) {
  if (l == r) return l;
  int leftnum = tree[tree[R].l].sum - tree[tree[L].l].sum;
  int mid = (l + r) >> 1;
  if (k <= leftnum) return Query(l, mid, tree[L].l, tree[R].l, k);
  else return Query(mid + 1, r, tree[L].r, tree[R].r, k - leftnum);
}

int main() {
  int n, m; scanf("%d%d", &n, &m);  //n个数,m个查询
  for (int i = 1; i <= n; i++) {
    scanf("%d", &a[i]);
    b[i] = a[i];
  }
  sort(b + 1, b + n + 1);
  int sz = unique(b + 1, b + n + 1) - (b + 1);  //离散后的大小
  for (int i = 1; i <= n; i++) {
    int inx = lower_bound(b + 1, b + sz + 1, a[i]) - b;
    Insert(1, sz, root[i - 1], root[i], inx);
  }
  while (m--) {
    int l, r, k; scanf("%d%d%d", &l, &r, &k);
    printf("%d\n", b[Query(1, sz, root[l - 1], root[r], k)]);
  }
  return 0;
}
```

## 树状数组套权值线段树

```cpp
//动态第k小
#include <bits/stdc++.h>
#define lowbit(x) ((x) & (-(x)))
using namespace std;

const int MAX = 1e5 + 50;
struct Node {
  int l, r, cnt;
}tree[MAX << 9];
int root[MAX], idx;
int a[MAX], n, m;
int X[MAX], Y[MAX], C1, C2;

void modify(int& k, int l, int r, int x, int y) {
  if (!k) k = ++idx;
  tree[k].cnt += y;
  if (l == r) return;
  int mid = (l + r) >> 1;
  if (x <= mid) modify(tree[k].l, l, mid, x, y);
  else modify(tree[k].r, mid + 1, r, x, y);
}

int query1(int l, int r, int k) {
  if (l == r) return l;
  int mid = (l + r) >> 1;
  int sum = 0;
  for (int i = 1; i <= C1; i++) sum -= tree[tree[X[i]].l].cnt;
  for (int i = 1; i <= C2; i++) sum += tree[tree[Y[i]].l].cnt;
  if (k <= sum){
    for (int i = 1; i <= C1; i++) X[i] = tree[X[i]].l;
    for (int i = 1; i <= C2; i++) Y[i] = tree[Y[i]].l;
    return query1(l, mid, k);
  }else{
    for (int i = 1; i <= C1; i++) X[i] = tree[X[i]].r;
    for (int i = 1; i <= C2; i++) Y[i] = tree[Y[i]].r;
    return query1(mid+1, r, k-sum);
  }
}

void add(int pos, int val, int x){
  for (int i = pos; i <= n; i += lowbit(i)) modify(root[i], 0, 1e9, val, x);
}

int query2(int L, int R, int k){
  C1 = C2 = 0;
  for (int i = L - 1; i >= 1; i -= lowbit(i)) X[++C1] = root[i];
  for (int i = R; i >= 1; i -= lowbit(i)) Y[++C2] = root[i];
  return query1(0, 1e9, k);
}

int main(){
  scanf("%d%d", &n, &m);
  for (int i = 1; i <= n; i++) {
    scanf("%d", &a[i]);
    add(i, a[i], 1);
  }
  for (int i = 1; i <= m; i++) {
    char op; scanf(" %c", &op);
    if (op == 'C') {
      int x, y; scanf("%d%d", &x, &y);
      add(x, a[x], -1);
      add(x, y, 1);
      a[x] = y;
    } else {
      int l, r, k; scanf("%d%d%d", &l, &r, &k);
      printf("%d\n", query2(l, r, k));
    }
  }
  return 0;
}
```

## 划分树

```cpp
//区间第k小
#include <bits/stdc++.h>
using namespace std;

const int MAX = 2e5 + 50;
int tree[20][MAX], toLeft[20][MAX];
int a[MAX], b[MAX], n, m;

void build(int deep, int l, int r) {
  if (l == r) return;
  int mid = (l + r) >> 1;
  int x = b[mid], same = mid - l + 1, posL = l, posR = mid + 1;
  for (int i = l; i <= r; i++) if (tree[deep][i] < x) --same;
  for (int i = l; i <= r; i++) {
    int flag = 0;
    if (tree[deep][i] < x || tree[deep][i] == x && same > 0) {
      flag = 1;
      tree[deep + 1][posL++] = tree[deep][i];
      same -= (tree[deep][i] == x);
    } else {
      tree[deep + 1][posR++] = tree[deep][i];
    }
    toLeft[deep][i] = toLeft[deep][i - 1] + flag;
  }
  build(deep + 1, l, mid);
  build(deep + 1, mid + 1, r);
}

int query(int deep, int l, int r, int L, int R, int k) {
  if (L == R) return tree[deep][L];
  int mid = (l + r) >> 1;
  int lx = toLeft[deep][L - 1] - toLeft[deep][l - 1];
  int ly = toLeft[deep][R] - toLeft[deep][l - 1];
  int rx = L - l - lx, ry = R - l + 1 - ly;
  int cnt = ly - lx;
  if (k <= cnt) return query(deep + 1, l, mid, l + lx, l + ly - 1, k);
  else return query(deep + 1, mid + 1, r, mid + rx + 1, mid + ry, k - cnt);
}

int main() {
  scanf("%d%d", &n, &m);
  for (int i = 1; i <= n; i++) {
    scanf("%d", &a[i]);
    tree[0][i] = b[i] = a[i];
  }
  sort(b + 1, b + n + 1);
  build(0, 1, n);
  while (m--) {
    int l, r, k; scanf("%d%d%d", &l, &r, &k);
    printf("%d\n", query(0, 1, n, l, r, k));
  }
  return 0;
}
```

## Splay

```cpp
//普通平衡树
#include <bits/stdc++.h>
#define INF 0x3f3f3f3f
using namespace std;

const int MAX = 1e6 + 50;
struct Node {
  int val;  //该点的值
  int cnt;  //该点在树上的出现次数
  int sz;  //该点子树大小(包括自己)
  int fa;  //该点父节点
  int son[2];  //该点左右儿子节点
}tree[MAX];
int root, TreeSize;

void Pushup(int x) {  //更新子树大小
  tree[x].sz = tree[tree[x].son[0]].sz + tree[tree[x].son[1]].sz + tree[x].cnt;
}

void Rotate(int x) {  //旋转该点
  int fa = tree[x].fa;  //x的父亲
  int gfa = tree[fa].fa;  //x父亲的父亲
  int k = (tree[fa].son[1] == x);  //x是父亲哪个儿子,0左儿子,1右儿子
  //Pushdown(x); Pushdown(fa);
  tree[gfa].son[tree[gfa].son[1] == fa] = x;  //x取代了父亲的位置
  tree[x].fa = gfa;
  tree[fa].son[k] = tree[x].son[k ^ 1];
  tree[tree[x].son[k ^ 1]].fa = fa;
  tree[x].son[k ^ 1] = fa;
  tree[fa].fa = x;
  Pushup(fa); Pushup(x);
}

void splay(int x, int goal) {  //将x旋转成goal的儿子,goal为0表示将x旋转到根
  while (tree[x].fa != goal) {
    int fa = tree[x].fa, gfa = tree[fa].fa;
    if (gfa != goal){
      ((tree[gfa].son[0]==fa) ^ (tree[fa].son[0]==x)) ? Rotate(x) : Rotate(fa);
    }
    Rotate(x);
  }
  if (!goal) root = x;  //goal为0,将根更新为x
}

void Find(int val) {  //查找val的位置并旋转到根
  if (!root) return;
  int u = root;
  while (val != tree[u].val && tree[u].son[val > tree[u].val]) {
    u = tree[u].son[val > tree[u].val];
  }
  splay(u, 0);
}

int Findkth(int k) {  //查找第k小
  if (tree[root].sz == 2) return -1;  //只有两个哨兵
  if (tree[root].sz < k) return -1;  //大于整个树的大小
  int u = root;
  while (1) {
    //Pushdown(u);
    if (k <= tree[tree[u].son[0]].sz) u = tree[u].son[0];
    else if (k <= tree[u].cnt+tree[tree[u].son[0]].sz) return tree[u].val;
    else {
      k -= tree[u].cnt + tree[tree[u].son[0]].sz;
      u = tree[u].son[1];
    }
  }
}

int PreSuf(int val, int f) {  //f=0寻找前驱,f=1寻找后继,返回val所在的位置
  Find(val);
  if (f && val < tree[root].val) return root;
  if (!f && val > tree[root].val) return root;
  int u = tree[root].son[f];
  if (!u) return 0;
  while (tree[u].son[f ^ 1]) u = tree[u].son[f ^ 1];
  return u;
}

void Insert(int val) {  //插入一个val
  int u = root, fa = 0;
  while (u && tree[u].val != val) {
    fa = u;
    u = tree[u].son[val > tree[u].val];  //val大于当前节点的值就往右,否则往左
  }
  if (u) {  //已经有val存在
    ++tree[u].cnt;
  } else {
    u = ++TreeSize;  //新节点位置
    if (fa) tree[fa].son[val > tree[fa].val] = u;
    tree[u].son[0] = tree[u].son[1] = 0;
    tree[u].val = val;
    tree[u].fa = fa;
    tree[u].cnt = tree[u].sz = 1;
  }
  //把当前位置移到根保证结构平衡,前面更改了子树大小必须splay去Pushup保证sz的正确
  splay(u, 0);
}

void Delete(int val) {  //删除一个val
  //先找到val的前驱和后继,然后把前驱splay到根,后继splay到前驱的儿子
  //此时val就是suf节点的左儿子,并且suf左儿子只有val这一个节点
  int pre = PreSuf(val, 0), suf = PreSuf(val, 1);
  splay(pre, 0);
  splay(suf, pre);
  int u = tree[suf].son[0];
  if (tree[u].cnt > 1) {
    --tree[u].cnt;
    splay(u, 0);
  } else {
    tree[suf].son[0] = 0;
  }
}

int main() {
  Insert(INF); Insert(-INF);  //先插入两个哨兵防止卡死
  int n; scanf("%d", &n);
  for (int i = 1; i <= n; i++) {
    int opt, x; scanf("%d%d", &opt, &x);
    if (opt == 1) Insert(x);
    if (opt == 2) Delete(x);
    if (opt == 3) {
      Find(x);
      printf("%d\n", tree[tree[root].son[0]].sz);
    }
    if (opt == 4) printf("%d\n", Findkth(x + 1));
    if (opt == 5) printf("%d\n", tree[PreSuf(x, 0)].val);
    if (opt == 6) printf("%d\n", tree[PreSuf(x, 1)].val);
  }
  return 0;
}
```

## 可持久化字典树

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 6e5 + 50;
int trie[MAX * 30][2], root[MAX], cnt[MAX * 30], idx;
int a[MAX], sum[MAX], n, m;

void Insert(int pre, int now, int x) {
  for (int i = 25; i >= 0; i--) {
    int t = (x >> i) & 1;
    trie[now][0] = trie[pre][0];
    trie[now][1] = trie[pre][1];
    trie[now][t] = ++idx;
    cnt[now] = cnt[pre] + 1;
    pre = trie[pre][t];
    now = trie[now][t];
  }
  cnt[now] = cnt[pre] + 1;
}

int query(int pre, int now, int x) {
  int res = 0;
  for (int i = 25; i >= 0; i--) {
    int t = (x >> i) & 1;
    if (cnt[trie[now][t ^ 1]] - cnt[trie[pre][t ^ 1]]) {
      res |= 1 << i;
      pre = trie[pre][t ^ 1];
      now = trie[now][t ^ 1];
    } else {
      pre = trie[pre][t], now = trie[now][t];
    }
  }
  return res;
}

int main() {
  scanf("%d%d", &n, &m); ++n;
  for (int i = 2; i <= n; i++) scanf("%d", &a[i]);
  for (int i = 1; i <= n; i++) {
    sum[i] = sum[i-1] ^ a[i];
    root[i] = ++idx;
    Insert(root[i-1], root[i], sum[i]);
  }
  while (m--) {
    char opt; scanf(" %c", &opt);
    if (opt == 'A') {
      scanf("%d", &a[++n]);
      sum[n] = sum[n - 1] ^ a[n];
      root[n] = ++idx;
      Insert(root[n - 1], root[n], sum[n]);
    } else {
      int l, r, x; scanf("%d%d%d", &l, &r, &x);
      printf("%d\n", query(root[l - 1], root[r], sum[n] ^ x));
    }
  }
  return 0;
}
```

## 手写hash

```cpp
namespace HASH {
  int to[MAX], cnt[MAX], nxt[MAX], head[MAX], tot;
  void init() { memset(head, tot = 0, sizeof(head)), memset(cnt, 0, sizeof(cnt)); }
  void add(int x) {
    int k = x % MOD;
    for (int i = head[k]; i; i = nxt[i]) if (to[i] == x) { ++cnt[i]; return; }
    to[++tot] = x, cnt[tot] = 1, nxt[tot] = head[k], head[k] = tot;
  }
  int query(int x) {
    int k = x % MOD;
    for (int i = head[k]; i; i = nxt[i]) if (to[i] == x) return cnt[i];
    return 0;
  }
}
```

## deque带翻转操作

```cpp
struct deQue {
  int buffer[MAX << 1];
  int head = MAX, tail = MAX - 1;
  bool rev;
  bool empty() { return tail < head; }
  int size() { return tail - head + 1; }
  int front() { return rev ? buffer[tail] : buffer[head]; }
  int back() { return rev ? buffer[head] : buffer[tail]; }
  void push_front(int x) { rev ? buffer[++tail] = x : buffer[--head] = x; }
  void push_back(int x) { rev ? buffer[--head] = x : buffer[++tail] = x; }
  void pop_back() { rev ? head++ : tail--; }
  void pop_front() { rev ? tail-- : head++; }
  void reverse() { rev ^= 1; }
};
```


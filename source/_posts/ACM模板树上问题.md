---
title: ACM模板树上问题
date: 2021/07/07 15:45:57
categories: 
- 模板
tags: 
- 模板
- 树上问题
---

<!--more-->

## 树上差分

```cpp
//树上边差分
d[u] += x;
d[v] += x;
d[LCA(u, v)] -= 2 * x;

//树上点差分
d[u] += x;
d[v] += x;
d[LCA(u, v)] -= x;
d[fa[LCA(u, v)]] -= x;
```

## RMQ+LCA

预处理：$O(VlogV)$，查询：$O(1)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 5e5 + 50;
struct Edge {
  int v, next;
}edges[MAX << 1];
int head[MAX], k, n, m, root;
int deep[MAX], first[MAX], oula[MAX << 1], idx;
int ST[21][MAX << 1], logn[MAX << 1];

void add(int u, int v) {
  edges[++k].next = head[u]; edges[k].v = v; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; head[v] = k;
}

void dfs(int u, int fa, int dep) {
  first[u] = ++idx;
  oula[idx] = u;
  deep[u] = dep;
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (v == fa) continue;
    dfs(v, u, dep + 1);
    oula[++idx] = u;
  }
}

void init_ST() {
  logn[1] = 0; logn[2] = 1;
  for (int i = 3; i <= idx; i++) logn[i] = logn[i >> 1] + 1;
  for (int i = 1; i <= idx; i++) ST[0][i] = oula[i];
  for (int j = 1; j < 21; j++) {
    for (int i = 1; i + (1 << j) - 1 <= idx; i++) {
      if (deep[ST[j - 1][i]] <= deep[ST[j - 1][i + (1 << (j - 1))]]) ST[j][i] = ST[j - 1][i];
      else ST[j][i] = ST[j - 1][i + (1 << (j - 1))];
    }
  }
}

int getLca(int u, int v) {
  int l = first[u], r = first[v];
  if (l > r) swap(l, r);
  int s = logn[r - l + 1];
  if (deep[ST[s][l]] <= deep[ST[s][r - (1 << s) + 1]]) return ST[s][l];
  else return ST[s][r - (1 << s) + 1];
}

int main() {
  scanf("%d%d%d", &n, &m, &root);
  for (int i = 1; i < n; i++) {
    int u, v; scanf("%d%d", &u, &v);
    add(u, v);
  }
  dfs(root, 0, 1);
  init_ST();
  while (m--) {
    int u, v; scanf("%d%d", &u, &v);
    printf("%d\n", getLca(u, v));
  }
  return 0;
}
```

## 点分治

复杂度：$O(Vlog^{2}V)$

一般处理树上路径询问之类的问题

```cpp
//树上路径 <= k 的数量
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1e4 + 50;
struct Edge {
    int v, w, next;
}edges[MAX << 1];
int head[MAX], k, n, kk;
int sz[MAX], d[MAX], rt, rtsz, mxsz;
int a[MAX], tot, ans;
bool vis[MAX];

void add(int u, int v, int w) {
  edges[++k].next = head[u]; edges[k].v = v; edges[k].w = w; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; edges[k].w = w; head[v] = k;
}

void getroot(int u, int fa) {
  sz[u] = 1;
  int mx = 0;
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (v == fa || vis[v]) continue;
    getroot(v, u);
    sz[u] += sz[v];
    mx = max(mx, sz[v]);
  }
  mx = max(mx, rtsz - sz[u]);
  if (mx < mxsz) rt = u, mxsz = mx;
}

void getdis(int u, int fa) {
  a[++tot] = d[u];
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (v == fa || vis[v]) continue;
    d[v] = d[u] + edges[i].w;
    getdis(v, u);
  }
}

int calc(int u, int x) {
  d[u] = x;
  tot = 0;
  getdis(u, 0);
  sort(a + 1, a + tot + 1);
  int l = 1, r = tot, res = 0;
  while (l < r) {
    while (l < r && a[l] + a[r] > kk) --r;
    res += r - l;
    ++l;
  }
  return res;
}

void dfs(int u) {
  vis[u] = true;
  ans += calc(u, 0);
  for (int i = head[u]; i; i = edges[i].next){
    int v = edges[i].v;
    if (vis[v]) continue;
    ans -= calc(v, edges[i].w);
    mxsz = INT_MAX; rtsz = sz[v];
    getroot(v, 0);
    dfs(rt);
  }
}

int main() {
  while (~scanf("%d%d", &n, &kk)) {
    if (n == 0 && kk == 0) break;
    for (int i = 0; i < n + 5; i++) head[i] = 0, vis[i] = false;
    k = 0;
    for (int i = 0; i < n - 1; i++) {
      int u, v, w; scanf("%d%d%d", &u, &v, &w);
      add(u, v, w);
    }
    ans = 0; rtsz = n; mxsz = INT_MAX;
    getroot(1, 0);
    dfs(rt);
    printf("%d\n", ans);
  }
  return 0;
}
```

## dsu on tree

一般用来解决大规模子树查询问题，复杂度$O(VlogV)$

算法流程大致如下：

1.预处理出每棵树的重儿子。

2.先统计轻儿子的值，再删除轻儿子的标记。

3.然后统计重儿子，不删除重儿子的标记。

4.并再次统计所有轻儿子，这颗子树就统计完成了。

```cpp
//CF600E 每个点有一种颜色,求每一颗子树最多颜色的编号的和
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int MAX = 1e5+10;
struct Edge{
  int v, next;
}edges[MAX << 1];
int head[MAX], k, n;
int sz[MAX], son[MAX], NowSon;
int col[MAX], cnt[MAX], Mx;
ll ans[MAX], tmp;

void add(int u, int v) {
  edges[++k].next = head[u]; edges[k].v = v; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; head[v] = k;
}

void dfs1(int u, int fa) {
  sz[u] = 1;
  son[u] = 0;
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (v == fa) continue;
    dfs1(v, u);
    sz[u] += sz[v];
    if (sz[v] > sz[son[u]]) son[u] = v;
  }
}

void Add(int u, int fa) {
  ++cnt[col[u]];
  if (cnt[col[u]] > 1LL * Mx) {
    Mx = cnt[col[u]];
    tmp = col[u];
  } else if (cnt[col[u]] == 1LL * Mx) tmp += col[u];
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (v == fa || v == NowSon) continue;
    Add(v, u);
  }
}

void Del(int u, int fa) {
  --cnt[col[u]];
  for (int i = head[u]; i; i = edges[i].next){
    int v = edges[i].v;
    if (v == fa) continue;
    Del(v, u);
  }
}

void dfs2(int u, int fa, bool keep) {
  for (int i = head[u]; i; i = edges[i].next){
    int v = edges[i].v;
    if (v == fa || v == son[u]) continue;
    dfs2(v, u, false);
  }
  if (son[u]) {
    dfs2(son[u], u, true);
    NowSon = son[u];
  }
  Add(u, fa);
  ans[u] = tmp;
  if (!keep) Del(u, fa), Mx = tmp = 0;
}

int main() {
  scanf("%d", &n);
  for (int i = 1; i <= n; i++) scanf("%d", &col[i]);
  for (int i = 0; i < n - 1; i++){
    int u, v; scanf("%d%d", &u, &v);
    add(u, v);
  }
  dfs1(1, 0);
  dfs2(1, 0, false);
  for (int i = 1; i <= n; i++) printf("%lld%c", ans[i], " \n"[i==n]);
  return 0;
}
```

## 树链剖分

```c++
#include <bits/stdc++.h>
#define ls k << 1
#define rs k << 1 | 1
#define lson k << 1, l, mid
#define rson k << 1 | 1, mid + 1, r
using namespace std;

const int INF = 0x3f3f3f3f
const int MAX = 3e4 + 50;
struct Node{
  int l, r, sum, Max;
}tree[MAX << 2];
struct Edge{
  int v, next;
}edges[MAX << 1];
int head[MAX], k, a[MAX];                   //两个dfs预处理以下信息
int fa[MAX], deep[MAX], sz[MAX], son[MAX];  //第一个:节点父亲、节点深度、节点大小、节点重儿子
int top[MAX], id[MAX], rk[MAX], idx;        //第二个:链顶节点、dfs序编号、编号对应的原节点

void add(int u, int v) {
  edges[++k].next = head[u]; edges[k].v = v; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; head[v] = k;
}
//-----------------------------预处理-----------------------------
void dfs1(int u, int pre, int dep) {
  fa[u] = pre;
  deep[u] = dep;
  sz[u] = 1;
  son[u] = 0;
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (v == pre) continue;
    dfs1(v, u, dep + 1);
    sz[u] += sz[v];
    if (sz[v] > sz[son[u]]) son[u] = v;
  }
}

void dfs2(int u, int utop) {
  top[u] = utop;
  id[u] = ++idx;
  rk[idx] = u;
  if (!son[u]) return;
  dfs2(son[u], utop);  //优先dfs重儿子,为了重链上的节点编号连续
  for (int i = head[u]; i; i = edges[i].next){
    int v = edges[i].v;
    if (v == fa[u] || v == son[u]) continue;
    dfs2(v, v);
  }
}
//----------------------------------------------------------------
//--------------------------线段树基本操作--------------------------
void pushup(int k) {
  tree[k].sum = tree[ls].sum + tree[rs].sum;
  tree[k].Max = max(tree[ls].Max, tree[rs].Max);
}

void Tbuild(int k, int l, int r) {
  tree[k].l = l; tree[k].r = r;
  if (l == r){
    tree[k].Max = tree[k].sum = a[rk[l]];  //赋上原节点的值
    return;
  }
  int mid = (l + r) >> 1;
  Tbuild(lson);
  Tbuild(rson);
  pushup(k);
}

void Tchange(int k, int x, int y) {
  if (tree[k].l == x && tree[k].r == x) {
    tree[k].Max = tree[k].sum = y;
    return;
  }
  int mid = (tree[k].l + tree[k].r) >> 1;
  if (x <= mid) Tchange(ls, x, y);
  else Tchange(rs, x, y);
  pushup(k);
}

void Tquery(int k, int l, int r, int& S, int& M) {
  if (tree[k].l == l && tree[k].r == r) {
    S += tree[k].sum;
    M = max(M, tree[k].Max);
    return;
  }
  int mid = (tree[k].l + tree[k].r) >> 1;
  if (r <= mid) Tquery(ls, l, r, S, M);
  else if (l > mid) Tquery(rs, l, r, S, M);
  else Tquery(lson, S, M), Tquery(rson, S, M);
}
//----------------------------------------------------------------
//------------------将树上的节点转化为链上的编号来操作-----------------
//慢慢感受
void change(int x, int y) {
  Tchange(1, id[x], y);
}

void query(int u, int v, int& S, int& M) {
  while (top[u] != top[v]) {
    if (deep[top[u]] < deep[top[v]]) swap(u, v);
    Tquery(1, id[top[u]], id[u], S, M);
    u = fa[top[u]];
  }
  if (deep[u] < deep[v]) swap(u, v);
  Tquery(1, id[v], id[u], S, M);
}
//----------------------------------------------------------------
int main() {
  int n; scanf("%d", &n);
  for (int i = 0; i < n - 1; i++) {
    int u, v; scanf("%d%d", &u, &v);
    add(u, v);
  }
  for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
  dfs1(1, 0, 1);
  dfs2(1, 1);
  Tbuild(1, 1, n);
  int q; scanf("%d", &q);
  while (q--) {
    char s[10];
    int x, y, S = 0, M = -INF;
    scanf("%s%d%d", s, &x, &y);
    if (!strcmp(s, "CHANGE")) change(x, y);
    else {
      query(x, y, S, M);
      if (!strcmp(s, "QMAX")) printf("%d\n", M);
      else printf("%d\n", S);
    }
  }
  return 0;
}
```

## 树上路径交

```cpp
int intersection(int x, int y, int xx, int yy) {
  int t[4] = { lca(x, xx), lca(x, yy), lca(y, xx), lca(y, yy) };
  sort(t, t + 4);
  int r = lca(x, y), rr = lca(xx, yy);
  if (dep[t[0]] < min(dep[r], dep[rr]) || dep[t[2]] < max(dep[r], dep[rr])) return 0;
  int tt = lca(t[2], t[3]);
  int ret = 1 + dep[t[2]] + dep[t[3]] - dep[tt] * 2;
  return ret;
}
```

## LCT

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1e5+50;
struct Node {
  int val;
  int sum;
  int fa;  //父亲
  int son[2];  //左右儿子
  bool rev;  //是否翻转
}tree[MAX];

void clean(int x) {  //清空节点
  tree[x].val = tree[x].sum = tree[x].rev = 0;
  tree[x].fa = tree[x].son[0] = tree[x].son[1] = 0;
}

int getson(int x) {  //获取x是父亲的哪个儿子
  return tree[tree[x].fa].son[1] == x;
}

int isroot(int x) {  //判断x是否为辅助树的根
  return tree[tree[x].fa].son[0] != x && tree[tree[x].fa].son[1] != x;
}

void pushup(int x) {  //更新标记
  tree[x].sum = tree[tree[x].son[0]].sum ^ tree[tree[x].son[1]].sum ^ tree[x].val;
}

void pushdown(int x) {
  if (tree[x].rev) {
    if (tree[x].son[0]) {
      tree[tree[x].son[0]].rev ^= 1;
      swap(tree[tree[x].son[0]].son[0], tree[tree[x].son[0]].son[1]);
    }
    if (tree[x].son[1]) {
      tree[tree[x].son[1]].rev ^= 1;
      swap(tree[tree[x].son[1]].son[0], tree[tree[x].son[1]].son[1]);
    }
    tree[x].rev = 0;
  }
}

void update(int x) {
  if (!isroot(x)) update(tree[x].fa);
  pushdown(x);
}

void rotate(int x) {
  int fa = tree[x].fa;
  int gfa = tree[fa].fa;
  int chx = getson(x), chy = getson(fa);
  tree[x].fa = gfa;
  if (!isroot(fa)) tree[gfa].son[chy] = x;
  tree[fa].son[chx] = tree[x].son[chx ^ 1];
  tree[tree[x].son[chx ^ 1]].fa = fa;
  tree[x].son[chx ^ 1] = fa;
  tree[fa].fa = x;
  pushup(fa); pushup(x); pushup(gfa);
}

void splay(int x) {
  update(x);
  for (int f = tree[x].fa; f = tree[x].fa, !isroot(x); rotate(x)) {
    if (!isroot(f)) rotate(getson(x) == getson(f) ? f : x);
  }
}

void access(int x) {  //将原树根到x的链设为实链
  for (int f = 0; x; f = x, x = tree[x].fa) {
    splay(x);
    tree[x].son[1] = f;
    pushup(x);
  }
}

void makeroot(int x) {  //使x成为原树的根
  access(x); splay(x);
  swap(tree[x].son[0], tree[x].son[1]);
  tree[x].rev ^= 1;
}

int findroot(int x) {  //寻找x的辅助树的根
  access(x); splay(x); pushdown(x);
  while (tree[x].son[0]) x = tree[x].son[0], pushdown(x);
  return x;
}

void split(int x, int y) {  //将x-y的路径放入同一splay中,此时y维护了链上信息
  makeroot(x); access(y); splay(y);
}

void link(int x, int y) {  //在x,y间连一条边
  makeroot(x);
  if (findroot(y) != x) tree[x].fa = y;
}

void cut(int x, int y) {  //将x,y间的边断开
  makeroot(x);
  if (findroot(y) == x && tree[x].fa == y && tree[y].son[0] == x && !tree[x].son[1]) {
    tree[x].fa = tree[y].son[0] = 0;
    pushup(y);
  }
}

int main() {
  int n, m; scanf("%d%d", &n, &m);
  for (int i = 1; i <= n; i++) scanf("%d", &tree[i].val);
  while (m--) {
    int op, x, y; scanf("%d%d%d", &op, &x, &y);
    if (op == 0) split(x, y), printf("%d\n", tree[y].sum);
    else if (op == 1) link(x, y);
    else if (op == 2) cut(x, y);
    else splay(x), tree[x].val = y;
  }
  return 0;
}
```
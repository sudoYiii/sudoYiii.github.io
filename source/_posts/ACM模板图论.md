---
title: ACM模板图论
date: 2021/07/07 15:40:57
categories: 
- 模板
tags: 
- 模板
- 图论
---

<!--more-->

Tarjan时间复杂度：$O(V+E)$

要注意原图不连通以及重边的情况

## Tarjan求割点

```c++
int dfn[MAX], low[MAX], idx;
bool is[MAX];
vector<int> edges[MAX];

void tarjan(int u, int root) {
  dfn[u] = low[u] = ++idx;
  int child = 0;
  for (auto& v: edges[u]) {
    if (dfn[v] == 0) {
      child++;
      tarjan(v, root);
      low[u] = min(low[u], low[v]);
      //两种情况1.根节点且孩子>=2;
      //       2.非根节点且有一个孩子能回溯的最早的low不超过自己dfn的值.
      if (u == root && child >= 2) is[u] = true;
      else if (u != root && low[v] >= dfn[u]) is[u] = true;
    } else {
      low[u] = min(low[u], dfn[v]);
    }
  }
}
```

## Tarjan求桥

```c++
struct Edge {
    int v, next;
}edges[MAX];
int head[MAX], k = 1, n, m;
int dfn[MAX], low[MAX], idx;
bool bridge[MAX];

void tarjan(int u, int pre) {
  dfn[u] = low[u] = ++idx;
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (i == (pre ^ 1)) continue;
    if (dfn[v] == 0) {
      tarjan(v, i);
      low[u] = min(low[u], low[v]);
      //桥的判断条件
      if (low[v] > dfn[u]) bridge[i] = bridge[i ^ 1] = true;
    } else {
      low[u] = min(low[u], dfn[v]);
    }
  }
}
```

## Tarjan缩点

```c++
int dfn[MAX], low[MAX], idx;
int stk[MAX], top;
int belong[MAX], sum;  //记录强连通分量的集合
int in[MAX], out[MAX];  //记录每个强连通分量的入度和出度
bool instk[MAX];

void tarjan(int u, int pre) {
  dfn[u] = low[u] = ++idx;
  stk[++top] = u;
  instk[u] = true;
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (i == (pre ^ 1)) continue;
    if (dfn[v] == 0) {
      tarjan(v, i);
      low[u] = min(low[u], low[v]);
    } else if (instk[v]) {
      low[u] = min(low[u], dfn[v]);
    }
  }
  if (dfn[u] == low[u]) {  //将n个点缩成sum个强连通分量
    int v;
    belong[u] = ++sum;
    instk[u] = false;
    do {
        v = stk[top--];
        belong[v] = sum;
        instk[v] = false;
    } while (u != v);
  }
}
```

## Dijkstra

时间复杂度：$O((V+E)logV)$

```c++
struct Edge {
  int v, w, next;
}edges[MAX];
int head[MAX], d[MAX], k, n, m;
bool vis[MAX];

void add(int u, int v, int w) {
  edges[++k].next = head[u]; edges[k].v = v; edges[k].w = w; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; edges[k].w = w; head[v] = k;
}

int dijkstra(int s, int t) {
  memset(vis, false, sizeof(vis));
  memset(d, 0x3f, sizeof(d));
  priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> q;
  q.push(P(0, s));
  d[s] = 0;
  while (q.size()) {
    int u = q.top().second; q.pop();
    if (vis[u]) continue;
    vis[u] = true;
    for (int i = head[u]; i; i = edges[i].next) {
      int v = edges[i].v;
      if (!vis[v] && d[v] > d[u] + edges[i].w) {
        d[v] = d[u] + edges[i].w;
        q.push(P(d[v], v));
      }
    }
  }
  if (d[t] == INF) return -1;
  return d[t];
}
```

## Floyd

时间复杂度：$O(V^3)$

```c++
int floyd() {
  for (int k = 0; k < n; k++)
    for (int i = 0; i < n; i++)
      for (int j = 0; j < n; j++)
        g[i][j] = min(g[i][j], g[i][k] + g[k][j]);
  return g[0][n - 1];
}
```

## SPFA

时间复杂度：$O(VE)$

```c++
struct Edge {
    int v, w, next;
}edges[MAX];
int head[MAX], k, n, m;
int d[MAX], cnt[MAX];
bool vis[MAX];

void add(int u, int v, int w) {
  edges[++k].next = head[u]; edges[k].v = v; edges[k].w = w; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; edges[k].w = w; head[v] = k;
}

int spfa(int s, int t) {
  memset(d, 0x3f, sizeof(d));
  memset(vis, false, sizeof(vis));
  queue<int> q;
  q.push(s);
  d[s] = 0;
  while (q.size()) {
    int u = q.front(); q.pop();
    vis[u] = false;
    for (int i = head[u]; i; i = edges[i].next) {
      int v = edges[i].v;
      if (d[v] > d[u] + edges[i].w) {
        d[v] = d[u] + edges[i].w;
        if (!vis[v]) {
          q.push(v);
          vis[v] = true;
          //下面这句可以用来判负环
          if (++cnt[v] > n) return -1;
        }
      }
    }
  }
  if (d[t] == INF) return -1;
  return d[t];
}
```

## Prim

时间复杂度：$O(V^2)$ (朴素版)

​		     $O(ElogV)$ (堆优化版)

```c++
int g[MAX][MAX], d[MAX], n, m;
bool vis[MAX];

int prim() {
  memset(vis, false, sizeof(vis));
  memset(d, 0x3f, sizeof(d));
  d[1] = 0;
  int sum = 0;
  for (int i = 1; i <= n; i++) {
    int mi = INF, k;
    for (int j = 1; j <= n; j++) {
      if (!vis[j] && d[j] < mi) mi = d[k = j];
    }
    if (mi == INF) break;
    vis[k] = true;
    sum += d[k];
    for (int j = 1; j <= n; j++) {
      if (!vis[j] && d[j] > g[k][j]) d[j] = g[k][j];
    }
  }
  return sum;
}
```

## Kruskal

时间复杂度：$O(ElogE)$

```c++
struct Edge {
  int u, v, w;
  bool operator < (const Edge& a)const {
    return w < a.w;
  }
}edges[MAX];
int fa[MAX], n, m;

int Find(int x) {
  if (fa[x] == x) return x;
  return fa[x] = Find(fa[x]);
}

bool Merge(int x, int y) {
  int rx = Find(x);
  int ry = Find(y);
  if (rx == ry) return false;
  fa[rx] = ry;
  return true;
}

int main() {
  scanf("%d%d", &n, &m);
  iota(fa, fa + n + 5, 0);
  for (int i = 0; i < m; i++) scanf("%d%d%d", &edges[i].u, &edges[i].v, &edges[i].w);
  sort(edges, edges + m);
  int sum = 0;
  for (int i = 0; i < m; i++) {
    if (Merge(edges[i].u, edges[i].v)) sum += edges[i].w;
  }
  printf("%d\n", sum);
  return 0;
}
```

一些性质：

最大匹配数：最大匹配的匹配边的数目

最小点覆盖数：选取最少的点，使任意一条边至少有一个端点被选择

最大独立数：选取最多的点，使任意所选两点均不相连

最小路径覆盖数：对于一个 DAG（有向无环图），选取最少条路径，使得每个顶点属于且仅属于一条路径。路径长可以为 0（即单个点）。

不存在奇数环

最小点覆盖 = 最大匹配数

最少路径点覆盖 = 点数 - 二分图的最大匹配

最大独立集 = 点数 - 二分图的最大匹配

## 匈牙利(最大匹配)

时间复杂度：$O(V^3)$  

```cpp
const int MAXN = 1e3+10, MAXM = 5e6+10;
struct Edge {
  int v, next;
}edges[MAXM];
int head[MAXN], k, n, m, e;
int match[MAXN];
bool vis[MAXN];

void add(int u, int v) {
  edges[++k].next = head[u]; edges[k].v = v; head[u] = k;
}

bool dfs(int u) {
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (vis[v]) continue;
    vis[v] = true;
    if (!match[v] || dfs(match[v])) {
      match[v] = u;
      return true;
    }
  }
  return false;
}

int hungary() {
  int res = 0;
  for (int i = 1; i <= n; i++) {
    memset(vis, false, sizeof(vis));
    if (dfs(i)) ++res;
  }
  return res;
}

int main() {
  scanf("%d%d%d", &n, &m, &e);
  for (int i = 0; i < e; i++) {
    int x, y; scanf("%d%d", &x, &y);
    if (x <= n && y <= m) add(x, y);
  }
  printf("%d\n", hungary());
  return 0;
}
```

## Hopcroft-Kcap(最大匹配)

时间复杂度：$O(\sqrt VE)$

```cpp
const int MAX = 3e3+10;
struct Edge {
  int v, next;
}edges[MAX * MAX];
int head[MAX], k;
int Mx[MAX], My[MAX], Nx, Ny;
int dx[MAX], dy[MAX], dis;
bool vis[MAX];

void add(int u, int v) {
  edges[++k].next = head[u]; edges[k].v = v; head[u] = k;
}

bool bfs() {
  memset(dx, 0x3f, sizeof(dx));
  memset(dy, 0x3f, sizeof(dy));
  dis = INF;
  queue<int> q;
  for (int i = 1; i <= Nx; i++) {
    if (!Mx[i]) {
      q.push(i);
      dx[i] = 0;
    }
  }
  while (q.size()) {
    int u = q.front(); q.pop();
    if (dx[u] > dis) break;
    for (int i = head[u]; i; i = edges[i].next) {
      int v = edges[i].v;
      if (dy[v] == INF) {
        dy[v] = dx[u] + 1;
        if (!My[v]) dis = dy[v];
        else {
          dx[My[v]] = dy[v] + 1;
          q.push(My[v]);
        }
      }
    }
  }
  return dis != INF;
}

bool dfs(int u) {
  for (int i = head[u]; i; i = edges[i].next) {
    int v = edges[i].v;
    if (!vis[v] && dy[v] == dx[u]+1) {
      vis[v] = true;
      if (My[v] && dy[v] == dis) continue;
      if (!My[v] || dfs(My[v])) {
        My[v] = u;
        Mx[u] = v;
        return true;
      }
    }
  }
  return false;
}

int HK() {
  memset(Mx, 0, sizeof(Mx));
  memset(My, 0, sizeof(My));
  int res = 0;
  while (bfs()) {
    memset(vis, false, sizeof(vis));
    for (int i = 1; i <= Nx; i++) {
      if (!Mx[i] && dfs(i)) ++res;
    }
  }
  return res;
}
```

## 最大流

最小割 $=$ 最大流

```cpp
const int INF = 0x3f3f3f3f;
const int MAXN = 250, MAXM = 5e3+50;
struct Edge {
  int v, next;
  int flow;
}edges[MAXM << 1];
int head[MAXN], cur[MAXN], k = 1;
int deep[MAXN], n, m, s, t;
queue<int> q;

void add(int u, int v, int flow) {
  edges[++k].next = head[u]; edges[k].v = v; edges[k].flow = flow; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; edges[k].flow = 0; head[v] = k;
}

bool bfs() {
  memset(deep, 0, sizeof(int) * (t + 5));
  while (q.size()) q.pop();
  q.push(s);
  deep[s] = 1;
  while (q.size()) {
    int u = q.front(); q.pop();
    for (int i = head[u]; i; i = edges[i].next) {
      int v = edges[i].v;
      if (!deep[v] && edges[i].flow > 0) {
        deep[v] = deep[u] + 1;
        q.push(v);
      }
    }
  }
  return deep[t] != 0;
}

int dfs(int u, int nowflow) {
  if (u == t) return nowflow;
  int flow = 0;
  for (int i = cur[u]; i; i = edges[i].next) {
    cur[u] = i;
    int v = edges[i].v;
    if (deep[v] == deep[u] + 1 && edges[i].flow > 0) {
      int tmp = dfs(v, min(nowflow, edges[i].flow));
      if (!tmp) continue;
      nowflow -= tmp;
      flow += tmp;
      edges[i].flow -= tmp;
      edges[i ^ 1].flow += tmp;
      if (!nowflow) break;
    }
  }
  return flow;
}

int Dinic() {
  int res = 0;
  while (bfs()) {
    for (int i = 1; i <= t; i++) cur[i] = head[i];
    res += dfs(s, INF);
  }
  return res;
}

int main() {
  scanf("%d%d%d%d", &n, &m, &s, &t);
  for (int i = 1; i <= m; i++) {
    int u, v, w; scanf("%d%d%d", &u, &v, &w);
    add(u, v, w);
  }
  printf("%d\n", Dinic());
  return 0;
}
```

## 最小费用最大流

```cpp
const int INF = 0x3f3f3f3f;
const int MAXN = 5e3+50, MAXM = 5e4+50;
struct Edge {
  int v, next;
  int flow, cost;
}edges[MAXM << 1];
int head[MAXN], cur[MAXN], k = 1;
int dis[MAXN], maxFlow, minCost;
int n, m, s, t;
bool vis[MAXN];
queue<int> q;

void add(int u, int v, int flow, int cost) {
  edges[++k].next = head[u]; edges[k].v = v; edges[k].flow = flow; edges[k].cost = cost; head[u] = k;
  edges[++k].next = head[v]; edges[k].v = u; edges[k].flow = 0; edges[k].cost = -cost; head[v] = k;
}

bool spfa() {
  memset(dis, 0x3f, sizeof(int) * (t + 5));
  while (q.size()) q.pop();
  q.push(s);
  dis[s] = 0;
  while (q.size()) {
    int u = q.front(); q.pop();
    vis[u] = false;
    for (int i = head[u]; i; i = edges[i].next) {
      int v = edges[i].v;
      if (edges[i].flow > 0 && dis[v] > dis[u] + edges[i].cost) {
        dis[v] = dis[u] + edges[i].cost;
        if (!vis[v]) {
          q.push(v);
          vis[v] = true;
        }
      }
    }
  }
  return dis[t] != INF;
}

int dfs(int u, int nowflow) {
  if (u == t) return nowflow;
  int flow = 0;
  vis[u] = true;
  for (int i = cur[u]; i; i = edges[i].next) {
    cur[u] = i;
    int v = edges[i].v;
    if (!vis[v] && edges[i].flow > 0 && dis[v] == dis[u] + edges[i].cost) {
      int tmp = dfs(v, min(nowflow, edges[i].flow));
      if (!tmp) continue;
      nowflow -= tmp;
      flow += tmp;
      edges[i].flow -= tmp;
      edges[i ^ 1].flow += tmp;
      minCost += edges[i].cost * tmp;
      if (!nowflow) break;
    }
  }
  vis[u] = false;
  return flow;
}

void Dinic() {
  while (spfa()) {
    for (int i = 1; i <= t; i++) cur[i] = head[i];
    maxFlow += dfs(s, INF);
  }
}

int main() {
  scanf("%d%d%d%d", &n, &m, &s, &t);
  for (int i = 1; i <= m; i++){
    int u, v, w, c; scanf("%d%d%d%d", &u, &v, &w, &c);
    add(u, v, w, c);
  }
  Dinic();
  printf("%d %d\n", maxFlow, minCost);
  return 0;
}
```

## Stoer_Wagner

求无向图最小割，时间复杂度：$O(VE+V^2logV)$。

```cpp
int n, m, g[MAX][MAX], dis[MAX];
bool vis[MAX], bin[MAX];

int contract(int& s, int& t) {
  memset(dis, 0, sizeof(dis));
  memset(vis, false, sizeof(vis));
  int minCut = 0;
  for (int i = 1; i <= n; i++) {
    int k = -1, maxc = -1;
    for (int j = 1; j <= n; j++) {
      if (!bin[j] && !vis[j] && dis[j] > maxc) {
        maxc = dis[j];
        k = j;
      }
    }
    if (k == -1) return minCut;
    s = t; t = k; minCut = maxc;
    vis[k] = true;
    for (int j = 1; j <= n; j++) {
      if (!bin[j] && !vis[j]) dis[j] += g[k][j];
    }
  }
  return minCut;
}

int Stoer_Wagner() {
  int minCut = INF;
  for (int i = 1, s, t; i < n; i++) {
    int ans = contract(s, t);
    bin[t] = true;
    if (ans < minCut) minCut = ans;
    if (minCut == 0) return 0;
    for (int j = 1; j <= n; j++) {
      if (!bin[j]) {
        g[j][s] += g[j][t];
        g[s][j] = g[j][s];
      }
    }
  }
  return minCut;
}
```

## 有源汇上下界最大流

```cpp

```


---
title: ACM模板字符串类
date: 2021/07/07 15:38:57
categories: 
- 模板
tags: 
- 模板
- 字符串
---

<!--more-->

## 字符串哈希

```cpp
struct StrHash {
  char s[MAX];
  int len;
  ull h[MAX], p[MAX], B;
  char& operator [] (const int& pos) { return s[pos]; }
  void init() {
    len = strlen(s + 1);
    B = 786433;
    p[0] = 1;
    for (int i = 1; i <= len; i++) {
      h[i] = h[i - 1] * B + s[i] - 'a';
      p[i] = p[i - 1] * B;
    }
  }
  ull calc(int l, int r) { return h[r] - h[l-1] * p[r - l + 1]; }
};
```

## KMP

时间复杂度：$O(n+m)$

```c++
char s[MAX], t[MAX];
int next[MAX], len_s, len_t;

void getNext() {
  int i = 0, k = -1;
  next[i] = k;
  while (i < len_t) {
    if (k == -1 || t[i] == t[k]) next[++i] = ++k;
    else k = next[k];
  }
}

int kmp() {
  getNext();
  int i = 0, j = 0;
  while (i < len_s && j < len_t) {
    if (j == -1 || s[i] == t[j]) ++i, ++j;
    else j = next[j];  //失配跳到next[j]的位置
    if (j == len_t) return i - j + 1;  //存在返回在主串中的位置,这里从1开始
  }
  return -1;  //不存在返回-1
}

int getMin() {  //最小表示法
  int i = 0, j = 1, k = 0;
  while (i < len_s && j < len_s && k < len_s) {
    int n = s[(i + k) % len_s] - s[(j + k) % len_s];
    if (n == 0) ++k;
    else {
      if (n > 0) i += k + 1;
      else j += k + 1;
      if (i == j) ++j;
      k = 0;
    }
  }
  return min(i, j);
}

int getMax() {  //最大表示法
  int i = 0, j = 1, k = 0;
  while (i < len_s && j < len_s && k < len_s){
    int n = s[(i + k) % len_s] - s[(j + k) % len_s];
    if (n == 0) ++k;
    else {
      if (n > 0) j += k + 1;
      else i += k + 1;
      if (i == j) ++j;
      k = 0;
    }
  }
  return min(i, j);
}
```

## Manacher

时间复杂度：$O(n)$

```c++
char s[MAX], str[MAX << 1];
int Len[MAX], len;

void init() {  //初始化
  int k = 0;
  str[k++] = '@'; str[k++] = '#';
  for (int i = 0; i < len; i++){
    str[k++] = s[i];
    str[k++] = '#';
  }
  str[k++] = '$';
  len = k;
}

int manacher() {
  init();
  int mx = 0, p = 0, ans = 0;
  for (int i = 0; i < len; i++) {
    if (i < mx) Len[i] = min(mx - i, Len[2 * p - i]);
    else Len[i] = 1;
    while (i - Len[i] >= 0 && str[i - Len[i]] == str[i + Len[i]]) ++Len[i];
    if (i + Len[i] > mx){
      mx = i + Len[i];
      p = i;
    }
    ans = max(ans, Len[i]);
  }
  return ans - 1;  //返回最长回文串的长度
}

int main() {
  scanf("%s", s);
  len = strlen(s);
  int ans = manacher();
  printf("%d\n", ans);
  return 0;
}
```

## Trie

```c++
int trie[MAX][26], End[MAX], idx;
char s[MAX];

void Insert(char s[]) {
  int p = 0, len = strlen(s);
  for (int i = 0; i < len; i++) {
    int c = s[i] - 'a';
    if (!trie[p][c]) trie[p][c] = ++idx;
    p = trie[p][c];
  }
  ++End[p];  //某个单词数量
}

int query(char s[]) {
  int p = 0, ans = 0, len = strlen(s);
  for (int i = 0; i < len; i++) {
    p = trie[p][s[i] - 'a'];
    if (p == 0) return ans;  //p==0说明字典中没有这个单词
    ans += End[p];
  }
  return ans;
}

int main() {
  int n; scanf("%d", &n);
  while (n--) {
    scanf("%s", s);
    Insert(s);  //往字典树中加入单词
  }
  int m; scanf("%d", &m);
  while (m--) {
    scanf("%s", s);
    printf("%d\n", query(s));
  }
  return 0;
}
```

## AC自动机+拓扑排序优化

```c++
int trie[MAX][26], fail[MAX], End[MAX], idx;
int cnt[MAX], ans[MAX], in[MAX], per[MAX];
char s[MAX];

int newNode() {
  memset(trie[idx], 0, sizeof(trie[idx]));
  fail[idx] = End[idx] = 0;
  return idx++;
}

void init() {
  idx = 0;
  newNode();
}

void insert(char s[], int id) {
  int p = 0, len = strlen(s);
  for (int i = 0; i < len; i++) {
    int c = s[i] - 'a';
    if (!trie[p][c]) trie[p][c] = newNode();
    p = trie[p][c];
  }
  if (End[p]) per[id] = End[p];
  else End[p] = id;
}

void build() {
  queue<int> q;
  for (int i = 0; i < 26; i++) if (trie[0][i]) q.push(trie[0][i]);
  while (q.size()) {
    int now = q.front(); q.pop();
    for (int i = 0; i < 26; i++) {
      if (trie[now][i]) {
        fail[trie[now][i]] = trie[fail[now]][i];
        q.push(trie[now][i]);
        ++in[fail[trie[now][i]]];
      } else {
        trie[now][i] = trie[fail[now]][i];
      }
    }
  }
}

void query(char s[]) {
  int p = 0, len = strlen(s);
  for (int i = 0; i < len; i++) {
    p = trie[p][s[i] - 'a'];
    ++cnt[p];
  }
}

void topu() {
  queue<int> q;
  for (int i = 1; i <= idx; i++) if (!in[i]) q.push(i);
  while (q.size()) {
    int u = q.front(); q.pop();
    ans[End[u]] = cnt[u];
    int v = fail[u];
    cnt[v] += cnt[u];
    --in[v];
    if (in[v] == 0) q.push(v);
  }
}

int main() {
  int n; scanf("%d", &n);
  iota(per, per + n + 5, 0); init();
  for (int i = 1; i <= n; i++) {
    scanf("%s", s);
    insert(s, i);
  }
  build();
  scanf("%s", s);
  query(s); topu();
  for (int i = 1; i <= n; i++) printf("%d\n", ans[per[i]]);
  return 0;
}
```

## 后缀数组

```cpp
struct SuffixArray {
  char s[MAX];
  int rk[MAX], y[MAX], tmp[MAX];
  int c[MAX], SA[MAX], Height[MAX], len, n, m;

  char& operator [] (const int& pos) { return s[pos]; }

  void Sort() {
    for (int i = 1; i <= m; i++) c[i] = 0;
    for (int i = 1; i <= n; i++) ++c[rk[i]];
    for (int i = 2; i <= m; i++) c[i] += c[i-1];
    for (int i = n; i >= 1; i--) SA[c[rk[y[i]]]--] = y[i];
  }

  void get_SA() {
    len = n = strlen(s + 1); m = 127;
    for (int i = 1; i <= n; i++) rk[i] = s[i] , y[i] = i;
    Sort();
    for (int k = 1; k <= n; k <<= 1) {
      int cnt = 0, num = 1;
      for (int i = n - k + 1; i <= n; i++) y[++cnt] = i;
      for (int i = 1; i <= n; i++) if (SA[i] > k) y[++cnt] = SA[i] - k;
      Sort(); swap(rk, tmp); rk[SA[1]] = 1;
      for (int i = 2; i <= n; i++) {
        if (tmp[SA[i]] == tmp[SA[i - 1]] && tmp[SA[i]+k] == tmp[SA[i - 1]+k]) rk[SA[i]] = num;
        else rk[SA[i]] = ++num;
      }
      m = num;
    }
  }

  void get_Hi() {  //SA[i]与SA[i - 1]的最长公共前缀
    int k = 0;
    for (int i = 1; i <= n; i++) {
      if (rk[i] == 1) continue;
      int j = SA[rk[i] - 1]; if (k) --k;
      while (i + k <= n && j + k <= n && s[i + k] == s[j + k]) ++k; 
      Height[rk[i]] = k;
    }
  }
};
```


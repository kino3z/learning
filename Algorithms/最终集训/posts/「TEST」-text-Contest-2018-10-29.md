title: 「TEST」Contest 2018 10 29
author: kririae
date: 2018-10-29 11:27:40
tags:
---
# Contest 2018 10 29

<!--more-->

> 解题报告

## $\text{T1}$

给出数列$A$，你需要自行选定$q$个长度为$k$的区间,每次删除区间内的最小值(当最小值有多个时任选一个),使得删除的$q$个数中最小值和最大值的差尽量小。$n \le 100000$。

毒瘤...
$\text{ARC098E}$，还带加强的。

原题数据范围是$O(n^2\log{n})$的，我的解法是这样的。
枚举删除数字的最小值，我们需要确定一个最小的最大值，从“限制”入手，枚举了最小值，代表所有比最小值小的都不能选了。确定最小的最大值有两种写法，比较类似。

可以二分最小的最大值，比最小值小的数字标记为$0$，最小值最大值之间的值标记为$1$，大于最大值的标记为$2$。所有含有$0$的区间都不能选，只能选仅含$1, 2$的区间。一个包含$1, 2$的区间中，又只有$1$才能对答案造成贡献，假如$1, 2$区间长度为$len$，我们只能从中选$len - k + 1$个$1$。贪心选择即可。

第二种写法不再介绍，相当类似，只是从区间中抽出最小的$len - k + 1$个数，放入新的数组排序，然后取第$q$个。

考虑$O(n\alpha(n))$的做法，考虑$\text{two pointers}$，对原数组排序，将$tpt$的左右指针放在排序后的数组上，我们向右移动指针的时候，顺带维护原数组的$0, 1, 2$性质。这样，每个数都会从$2 \rightarrow 1 \rightarrow 0$，当一个数从$1 \rightarrow 0$的时候，会将左右两个区间分开，我们会发现分开区间并不好处理，考虑合并区间。我们$tpt$倒过来扫，每个数字都会$0 \rightarrow 1 \rightarrow 2$，这样，就从分裂变成了合并。每当一个数字从$0 \rightarrow 1$的时候，就会合并其左右的区间，并且用并查集维护区间$1, 2$的个数。

这题的分析本来就不简单，拆分转合并的思路很妙，好题！

$AC$代码懒得写了，给$70$的吧。
```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int N = 2005;
int n, q, k;
ll a[N], b[N], c[N], cnt, tot;
int main() {
  scanf("%d%d%d", &n, &k, &q);
  for (int i = 1; i <= n; ++i)
    scanf("%lld", &a[i]);
  ll ans = 1ll << 60;
  for (int i = 1; i <= n; ++i) {
    tot = 0;
    a[n + 1] = -(1ll << 60);
    for (int j = 1; j <= n + 1; ++j)
      if (a[j] >= a[i])
        b[++cnt] = a[j];
      else {
        sort(b + 1, b + 1 + cnt);
        for (int i = 1; i <= cnt - k + 1; ++i)
          c[++tot] = b[i];
        cnt = 0;
      }
    sort(c + 1, c + 1 + tot);
    if (tot >= q)
      ans = min(ans, c[q] - a[i]);
  }
  cout << ans;
}
```

$\text{std}$:

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e5 + 10;
int n, dk, q, vn, a[N], nmax;
struct st {
  int v, pos;
} s[N];
bool cmp1(st a, st b) { return a.v > b.v; }
int v[N], fa[N], t[N], siz[N];
int find(int x) { return fa[x] == x ? x : fa[x] = find(fa[x]); }
void update(int k, int f) { nmax += f * max(0, min(v[k], siz[k] - dk + 1)); }
void insert(int k) {
  v[k] = t[k] = siz[k] = 1;
  if (t[k - 1]) {
    update(find(k - 1), -1);
    siz[k] += siz[fa[k - 1]], v[k] += v[fa[k - 1]], fa[fa[k - 1]] = k;
  }
  if (t[k + 1]) {
    update(find(k + 1), -1);
    siz[k] += siz[fa[k + 1]], v[k] += v[fa[k + 1]], fa[fa[k + 1]] = k;
  }
  update(find(k), 1);
}
void erase(int k) {
  update(find(k), -1);
  v[fa[k]]--;
  update(fa[k], 1);
  t[k] = 2;
}
bool judge() {
  int l, r = 0;
  nmax = 0;
  for (int i = 1; i <= n; i++) fa[i] = i, v[i] = t[i] = 0;
  for (l = 1; l <= n; erase(s[l++].pos)) {
    while (s[l].v - s[r + 1].v <= vn && r < n) {
      insert(s[++r].pos);
    }
    if (nmax >= q) return 1;
  }
  return 0;
}
int main() {
  scanf("%d%d%d", &n, &dk, &q);
  int l = 0, r = 0;
  for (int i = 1; i <= n; i++) {
    scanf("%d", &a[i]);
    r = max(r, a[i]);
  }
  for (int i = 1; i <= n; i++) s[i] = (st){a[i], i};
  sort(s + 1, s + n + 1, cmp1);
  while (l < r) {
    vn = (l + r) >> 1;
    if (judge())
      r = vn;
    else
      l = vn + 1;
  }
  while (!judge()) {
    vn++;
  }
  while (1 && vn > 1) {
    vn--;
    if (!judge()) {
      vn++;
      break;
    }
  }
  printf("%d", vn);
  return 0;
}
```

## $\text{T2}$

$n$点$m$边无向带权图。$s$个加油站，有$q$次询问，给出起点，终点，起点终点都是加油站，给出一辆车，车有油量上限$k_i$，油量满的情况下能行驶$k_i$的距离，问能否从$s \rightarrow t$。$n, s, q \le 100000$。

$\text{BZOJ4144}$
考场想到$50pts$，结果$4k$的代码写炸了...
图上查询问题，离不了树上问题的处理。
观察到，如果能安全从$s \rightarrow t$，必定会经过一些加油站。而且，对于经过的加油站，两两之间的距离必须$\le k$。暴力的话，考虑$s$两两之间的最短路建成完全图。在完全图上跑最小生成树，就能$s \rightarrow t$全部经过$\le k$的边。如果$s \rightarrow t$的最大边权$> k$，就不能从$s$到$t$。

然后我写炸了，然后爆零了。
瓶颈出来了，$s$的完全图的最小生成树怎么处理。
考虑图中的这种路径，$s, k, t$都是加油站，并且存在$s \rightarrow k$，$k \rightarrow t$，那么$s \rightarrow t$一定没有存在的必要。怎么实现呢？考虑$s \rightarrow k$上的非关键点，将这些非关键点分为两部分，到$s$最近的，到$k$最近的。我们设$from[i]$表示离$i$最近的关键点的编号，$dis[i]$表示$i$到$from[i]$的最短路。对于原图中的$u \rightarrow v$，$dis[u] + w + dis[v]$就是某两个关键点中的唯一距离，过滤掉经过其他点的。稍微解释一下，我就卡这里了：$dis[u]$一定不会经过任意一个加油站，如果经过的话，$dis[u]$就是当前点到经过的加油站的距离。$dis[v]$同理跑$dij$，把加油站全部扔进$\text{priority\_queue}$。就可以初始化$from$和$dis$数组了。对于图不连通，可以一开始并查集维护一发，也可以建虚边，看个人喜好。

代码：

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

inline char gc() {
  static char buf[1 << 18], *fs, *ft;
  return (fs == ft && (ft = (fs = buf) + fread(buf, 1, 1 << 18, stdin)), fs == ft) ? EOF : *fs++;
}
inline int read() {
  register int k = 0, f = 1;
  register char c = gc();
  for (; !isdigit(c); c = gc())
    if (c == '-') f = -1;
  for (; isdigit(c); c = gc()) k = (k << 3) + (k << 1) + (c - '0');
  return k * f;
}
const int N = 2e5 + 5;
int n, s, m, c[N];
int head[N], ver[N << 1], nxt[N << 1], edge[N << 1], tot;
int vis[N], from[N], fa[N];
ll dis[N];
inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
inline void addedge(int u, int v, int w) {
  ver[tot] = v;
  edge[tot] = w;
  nxt[tot] = head[u];
  head[u] = tot++;
}
namespace MST {
struct Edge {
  int u, v, w;
  bool operator<(const Edge &e) const { return w < e.w; }
} a[N << 1];
int head[N], ver[N << 1], nxt[N << 1], edge[N << 1], tot, cnt, fa[N];
int vis[N], f[N][30], g[N][30], dep[N];
inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
inline void addedge(int u, int v, int w) {
  ver[tot] = v;
  edge[tot] = w;
  nxt[tot] = head[u];
  head[u] = tot++;
}
inline void dfs(int x) {
  vis[x] = 1;
  for (int i = head[x], to; ~i; i = nxt[i])
    if (!vis[to = ver[i]]) {
      dep[to] = dep[x] + 1;
      f[to][0] = x;
      g[to][0] = edge[i];
      dfs(to);
    }
}
inline void init() {
  for (int i = 1; i <= n; ++i) fa[i] = i;
  memset(head, -1, sizeof head);
  tot = 0;
  sort(a + 1, a + 1 + cnt);
  for (int i = 1; i <= cnt; ++i) {
    int x = find(a[i].u), y = find(a[i].v);
    if (x != y) {
      fa[x] = y;
      addedge(a[i].u, a[i].v, a[i].w);
      addedge(a[i].v, a[i].u, a[i].w);
      // cout << "Add: " << a[i].u << " " << a[i].v << " " << a[i].w << endl;
    }
  }
  for (int i = 1; i <= s; ++i)
    if (!vis[c[i]]) dfs(c[i]);
  for (int t = 1; t <= 25; ++t)
    for (int i = 1, x; i <= s; ++i) {
      f[x = c[i]][t] = f[f[x][t - 1]][t - 1];
      g[x][t] = max(g[x][t - 1], g[f[x][t - 1]][t - 1]);
    }
}
inline int query(int x, int y) {
  if (dep[x] < dep[y]) swap(x, y);
  int ans = 0;
  for (int t = 25; t >= 0; --t)
    if (dep[f[x][t]] >= dep[y])
      ans = max(ans, g[x][t]), x = f[x][t];
  if (x == y) return ans;
  for (int t = 25; t >= 0; --t)
    if (f[x][t] != f[y][t]) {
      ans = max(ans, g[x][t]);
      ans = max(ans, g[y][t]);
      x = f[x][t], y = f[y][t];
    }
  ans = max(ans, g[x][0]);
  ans = max(ans, g[y][0]);
  return ans;
}
}  // namespace MST
inline void init() {
  static queue<int> q;
  while (!q.empty()) q.pop();
  memset(dis, 0x3f, sizeof dis);
  for (int i = 1; i <= s; ++i) {
    from[c[i]] = c[i];
    dis[c[i]] = 0;
    vis[c[i]] = 1;
    q.push(c[i]);
  }
  while (!q.empty()) {
    int u = q.front();
    q.pop();
    vis[u] = 0;
    for (int i = head[u], to; ~i; i = nxt[i])
      if (dis[to = ver[i]] > dis[u] + edge[i]) {
        dis[to] = dis[u] + edge[i];
        from[to] = from[u];
        if (!vis[to]) vis[to] = 1, q.push(to);
      }
  }
  for (int i = 1; i <= n; ++i)
    for (int j = head[i], to; ~j; j = nxt[j])
      if (from[i] != from[to = ver[j]]) {
        MST::a[++MST::cnt].u = from[i];
        MST::a[MST::cnt].v = from[to];
        MST::a[MST::cnt].w = dis[i] + dis[to] + edge[j];
      }
}
int main() {
  memset(head, -1, sizeof head);
  n = read(), s = read(), m = read();
  for (int i = 1; i <= s; ++i) c[i] = read();
  for (int i = 1; i <= n; ++i) fa[i] = i;
  for (int i = 1, u, v, w; i <= m; ++i) {
    u = read(), v = read(), w = read();
    addedge(u, v, w);
    addedge(v, u, w);
    int x = find(u), y = find(v);
    if (x != y) fa[x] = y;
  }
  init();
  MST::init();
  int x, y, b, q;
  q = read();
  while (q--) {
    x = read(), y = read(), b = read();
    if (find(x) == find(y)) {
      if (MST::query(x, y) > b)
        puts("NIE");
      else
        puts("TAK");
    } else
      puts("NIE");
  }
}
```

## $\text{T3}$

给出$n$个函数$f$，每个函数$f_i = \sum_{j = l_i}^{r_i}{a_j}$。作$a$的单点修改，查询$f$的区间和$f_l \rightarrow f_r$。

考虑暴力，对$a$维护$BIT$，作单点修改和区间和查询。每次访问$f$的区间和时，暴力求$f_l \rightarrow f_r$。查询复杂度$O(n\log{n})$，修改复杂度$O(\log{n})$。我们发现复杂度并不平衡，考虑用分块综合复杂度，具体来说，将$f$分为$\sqrt{n}$块，在块内对$f$按照$r$排序，并且求出差分数组，表示每个位置被$f$覆盖了多少次。对块进行整体修改时直接统计当前位置被覆盖的次数，对块进行部分修改时用之前的暴力。
代码没有。
{}
date: 2018-07-14 07:29:21
---
# 「学习笔记」Tarjan算法

## Tarjan算法简介
咕咕咕~
2018 - 7 - 13
尊从不咕咕咕的信念，我来补了
Tarjan算法，求点双边双割点桥等等...
然而这里我要说的是SCC...即`Strong Connected Component`。
Tarjan算法定义了两个很关键的数组，`dfn[]`, `low[]`，一个很关键的栈`s`。
`dfn`的定义如下，dfs序的时间戳。对于每个节点，其父节点的dfn值小于其本身。
`s`没有定义，指当前搜索树上走过的顺序。这个一会儿会细细解释。
`low`的定义如下~追溯值，顾名思义，能够追溯到一个点。`low[x]`表示其`x`节点其本身加上其子节点，能够从一条有向边出发，到达之前的栈`s`中的某个节点的，且这个节点拥有最小时间戳。
存在情况`dfn[x] == low[x]`。表示能够最早追溯到的点是`x`本身。感性理解一下，其本身以及子树不能通过任意一条边到其父亲节点，只能通过一条边到其本身，即构成一个环。那么，这个环上的节点一定处于一个`SCC`中。感性理解，这个环上的节点正在`s`中。  
定义后向边`(x, y)`。表示搜索树之外的边。对于任意一个边`(x, y)`。如果`y`没被访问过，则这条边处于搜索树上，即可以`low[x] = min(low[x], low[y])`。如果`y`被访问过，且`vis[y]`（y在栈中）。根据刚才的定义`low`。（`low[x]`表示其`x`节点其本身加上其子节点，能够从一条有向边出发，到达之前的栈`s`中的某个节点的，且这个节点拥有最小时间戳。）`low[x] = min(low[x], dfn[x])`。
伪代码如下
```fake
function Tarjan(x) 
	dfn[x] = low[x] = ++dfs_times 
	stack.push(x) in_stack[x] = true
	for e in E
		if !dfn[e.to]
			Tarjan(e.to)
			low[x] = min(low[x], low[e.to])
		else if in_stack[e.to]
			low[x] = min(low[x], dfn[e.to])
	if low[x] == dfn[x]
		do
			top = stack.top()
			stack.pop()
			in_stack[top] = 0
			// ... TODO
		while top != x			
```

## 「NOIP2009」信息传递

统计强连通分量里的最大size

```cpp
#define R register
#define ll long long
#include <bits/stdc++.h>

using namespace std;

const int maxn = 2e5 + 5;

vector<int> edges[maxn];
int n, dfn[maxn], low[maxn], cnt, ans = maxn, x;
bitset<maxn> vis;
stack<int> s;

inline void tarjan(int x)
{
  // 再打一次~
  low[x] = dfn[x] = ++cnt;
  s.push(x), vis[x] = 1;
  for (R int i = 0; i < edges[x].size(); ++i)
  {
    int to = edges[x][i];
    if(!dfn[to])
    {
      // 如果to在dfs树上
      tarjan(to);
      low[x] = min(low[x], low[to]);
    } else if(vis[to]) {
      // 如果to不在dfs树上，用dfn更新low[x]
      low[x] = min(low[x], dfn[to]);
    }
  }
  if(low[x] == dfn[x]) 
  {
    // 又到了pop的季节
    int cnt = 0;
    for (; ;)
    {
      int curr = s.top();
      s.pop(), ++cnt, vis[curr] = 0;
      if(curr == x) break;
    } // 把需要pop的都pop出来
    if(cnt > 1) ans = min(ans, cnt); // 如果不光是这条边的话
  }
}

int main()
{
  scanf("%d", &n);
  for (R int i = 1; i <= n; ++i)
  { 
    scanf("%d", &x);
    edges[i].push_back(x);
  }
  for (R int i = 1; i <= n; ++i)
    if(!dfn[i]) tarjan(i);
  printf("%d", ans);
}
```

## 「BZOJ1051」受欢迎的牛

缩点后统计出度为0的个数，然后输出连通图的size

```cpp
#define ll long long
#define R register
#include <bits/stdc++.h>

using namespace std;

const int maxn = 10005;

namespace IO
{
inline char gc()
{
  static char buf[1 << 18], *fs, *ft;
  return (fs == ft && (ft = (fs = buf) + fread(buf, 1, 1 << 18, stdin)), fs == ft) ? EOF : *fs++;
}
inline int read()
{
  register int k = 0, f = 1;
  register char c = gc();
  for (; !isdigit(c); c = gc()) if (c == '-') f = -1;
  for (; isdigit(c); c = gc()) k = (k << 3) + (k << 1) + (c - '0');
  return k * f;
}
}

namespace BZOJ1051
{
vector<int> edges[maxn];
bitset<maxn> vis;
stack<int> s;
int n, m, x, y, dfn[maxn], low[maxn], cnt, bl[maxn], bcnt, out[maxn], siz[maxn];

inline void tarjan(int x)
{
  dfn[x] = low[x] = ++cnt;
  s.push(x), vis[x] = 1;
  for (R int i = 0; i < edges[x].size(); ++i)
  {
    int to = edges[x][i];
    if(!dfn[to])
    {
      tarjan(to);
      low[x] = min(low[x], low[to]);
    } else if(vis[to]) {
      low[x] = min(low[x], dfn[to]);
    }
  }
  if(low[x] == dfn[x]) 
  {
    ++bcnt;
    for (; ; ) {
      int now = s.top();
      s.pop(), vis[now] = 0;
      bl[now] = bcnt; ++siz[bcnt];
      if(now == x) break; 
    }
  }
}

inline void solve()
{
  using namespace IO;
  n = read(), m = read(); 
  for (R int i = 1; i <= m; ++i)
  { 
    x = read(), y = read();
    edges[x].push_back(y);
  }
  for (R int i = 1; i <= n; ++i)
    if(!dfn[i]) tarjan(i);
  for (R int i = 1; i <= n; ++i)
    for (R int j = 0; j < edges[i].size(); ++j)
      // 遍历所有的边~
      if(bl[i] != bl[edges[i][j]]) ++out[bl[i]];
  int pos = 0;
  for (R int i = 1; i <= bcnt; ++i)
    if(out[i] == 0) 
      if(pos) return puts("0"), void();
      else pos = i;
    printf("%d", siz[pos]);
}
}

int main()
{
  return BZOJ1051::solve(), 0;
}
```

## 「BZOJ1179」Atm

缩点，记录块的权值和，然后在缩点上跑DAG最长路

```cpp
// by kririae
#define R register
#define ll long long
#include <bits/stdc++.h>

using namespace std;

const int maxn = 500005;
namespace IO
{
inline char gc()
{
  static char buf[1 << 18], *fs, *ft;
  return (fs == ft && (ft = (fs = buf) + fread(buf, 1, 1 << 18, stdin)), fs == ft) ? EOF : *fs++;
}
inline int read()
{
  register int k = 0, f = 1;
  register char c = gc();
  for (; !isdigit(c); c = gc()) if (c == '-') f = -1;
  for (; isdigit(c); c = gc()) k = (k << 3) + (k << 1) + (c - '0');
  return k * f;
}
}

namespace BZOJ1179
{
vector<int> edges[maxn], dag[maxn];
stack<int> S;
bitset<maxn> vis;
queue<int> q;
int n, m, x, y, s, p, cnt, tot;
int a[maxn], b[maxn], dfn[maxn], low[maxn], belong[maxn], sum[maxn], f[maxn];

inline void tarjan(int x)
{
  dfn[x] = low[x] = ++cnt;
  S.push(x), vis[x] = 1;
  for (R int i = 0; i < edges[x].size(); ++i)
  {
    int to = edges[x][i];
    if(!dfn[to])
      tarjan(to), low[x] = min(low[x], low[to]);
    else if(vis[to]) low[x] = min(low[x], dfn[to]);
  }
  if(low[x] == dfn[x])
  {
    int curr; ++tot;
    do {
      curr = S.top(); S.pop(), vis[curr] = 0;
      belong[curr] = tot, sum[tot] += a[curr];
    } while(curr != x);
  }
}

inline void BFS(int s)
{
  // DAG拓扑序转移，SPFA有点大材小用了...
  vis.reset(), vis[s] = 1;
  q.push(s), f[s] = sum[s];
  while(!q.empty())
  {
    int curr = q.front(); q.pop(); vis[curr] = 0;
    for (R int i = 0; i < dag[curr].size(); ++i)
    {
      int to = dag[curr][i];
      if(f[curr] + sum[to] > f[to])
      {
        f[to] = f[curr] + sum[to];
        if(!vis[to]) vis[to], q.push(to);
      }
    }
  }
}

inline void solve()
{
  using namespace IO;
  n = read(), m = read();
  for (R int i = 1; i <= m; ++i)
    x = read(), y = read(), edges[x].push_back(y);
  for (R int i = 1; i <= n; ++i)
    a[i] = read();
  s = read(), p = read();
  for (R int i = 1; i <= p; ++i)
    b[i] = read();
  // init
  // 其他的都不管了，我们先把点缩好~
  for (R int i = 1; i <= n; ++i)
    if(!dfn[i]) tarjan(i);
  // 缩好之后建DAG...跑dp
  for (R int i = 1; i <= n; ++i)
    for (R int j = 0; j < edges[i].size(); ++j)
    {
      int to = edges[i][j];
      if(belong[i] != belong[to]) 
        dag[belong[i]].push_back(belong[to]);
    }
  BFS(belong[s]);
  int ans = 0;
  for (R int i = 1; i <= p; ++i)
    ans = max(ans, f[belong[b[i]]]);
  printf("%d", ans);
}
}

int main()
{
  return BZOJ1179::solve(), 0;
}
```

## 「BZOJ2427」软件安装

tarjan缩点成森林，建虚点连边，然后进行背包树形dp

```cpp
// by kririae
#define R register
#define ll long long
#include <bits/stdc++.h>

using namespace std;

const int maxn = 105;

namespace IO
{
inline char gc()
{
  static char buf[1 << 18], *fs, *ft;
  return (fs == ft && (ft = (fs = buf) + fread(buf, 1, 1 << 18, stdin)), fs == ft) ? EOF : *fs++;
}
inline int read()
{
  register int k = 0, f = 1;
  register char c = gc();
  for (; !isdigit(c); c = gc()) if (c == '-') f = -1;
  for (; isdigit(c); c = gc()) k = (k << 3) + (k << 1) + (c - '0');
  return k * f;
}
}

namespace BZOJ2427
{
vector<int> edges[maxn], DAG[maxn];
stack<int> s;
int n, m, w[maxn], v[maxn], d[maxn], f[maxn][505];
int dfn[maxn], low[maxn], vis[maxn], cnt;
int belong[maxn], ws[maxn], vs[maxn], in[maxn], tot;

inline void tarjan(int x)
{
  dfn[x] = low[x] = ++cnt;
  vis[x] = 1, s.push(x);
  for (R int i = 0; i < edges[x].size(); ++i)
  {
    int to = edges[x][i];
    if(!dfn[to])
      tarjan(to), low[x] = min(low[x], low[to]);
    else if(vis[to]) 
      low[x] = min(low[x], dfn[to]);
  }
  if(low[x] == dfn[x])
  {
    int curr; ++tot;
    do {
      curr = s.top(), s.pop(), vis[curr] = 0;
      belong[curr] = tot, ws[tot] += w[curr], vs[tot] += v[curr];
    } while(curr != x);
  }
}

inline void dp(int x)
{
  for (R int i = ws[x]; i <= m; ++i) f[x][i] = vs[x];
  for (R int i = 0; i < DAG[x].size(); ++i)
  {
    int to = DAG[x][i];
    dp(to);
    for (R int j = m - ws[x]; j >= 0; --j) // 树形背包的板子...用就对了...
      for (R int k = 0; k <= j; ++k)
        f[x][j + ws[x]] = max(f[x][j + ws[x]], f[x][j + ws[x] - k] + f[to][k]);
  }
}

inline void solve()
{
  using namespace IO;
  n = read(), m = read();
  for (R int i = 1; i <= n; ++i) w[i] = read();
  for (R int i = 1; i <= n; ++i) v[i] = read();
  for (R int i = 1; i <= n; ++i) if(d[i] = read()) edges[d[i]].push_back(i);
  for (R int i = 1; i <= n; ++i)
    if(!dfn[i]) tarjan(i);
  for (R int i = 1; i <= n; ++i)
    for (R int j = 0; j < edges[i].size(); ++j)
    {
      int to = edges[i][j];
      if(belong[i] != belong[to]) ++in[belong[to]], DAG[belong[i]].push_back(belong[to]);
    }
  // for (R int i = 1; i <= tot; ++i) cout << ws[i] << " " << vs[i] << endl;
  for (R int i = 1; i <= tot; ++i)
    if(in[i] == 0) DAG[tot + 1].push_back(i); // 建虚拟节点统一森林，方便树形dp...
  dp(tot + 1);
  // cout << ws[tot + 1] << endl;
  printf("%d", f[tot + 1][m]);
}
}

int main()
{
  return BZOJ2427::solve(), 0;
}
```

## 「USACO08DEC」Trick

进行Tarjan缩点，
```cpp
// QwQ
// by kririae~
#define R register
#define ll long long
#include <bits/stdc++.h>

using namespace std;

const int maxn = 100005;

namespace IO
{
inline char gc()
{
  static char buf[1 << 18], *fs, *ft;
  return (fs == ft && (ft = (fs = buf) + fread(buf, 1, 1 << 18, stdin)), fs == ft) ? EOF : *fs++;
}
inline int read()
{
  register int k = 0, f = 1;
  register char c = gc();
  for (; !isdigit(c); c = gc()) if (c == '-') f = -1;
  for (; isdigit(c); c = gc()) k = (k << 3) + (k << 1) + (c - '0');
  return k * f;
}
}

namespace USACO08Trick
{
int n, nxt[maxn], dfn[maxn], low[maxn], times;
int tot, siz[maxn], bel[maxn];
vector<int> edges[maxn];
stack<int> s;
bitset<maxn> vis;

inline void addedge(int from, int to)
{
  edges[from].push_back(to);
}

inline void tarjan(R int x)
{
  low[x] = dfn[x] = ++times;
  s.push(x), vis[x] = 1;
  for (R int i = 0; i < edges[x].size(); ++i)
  {
    R int to = edges[x][i];
    if(!dfn[to])
      tarjan(to), low[x] = min(low[x], low[to]);
    else if(vis[to]) low[x] = min(low[x], dfn[to]);
  }
  if(low[x] == dfn[x])
  {
    R int curr; ++tot;
    do {
      curr = s.top(), s.pop(), vis[curr] = 0;
      bel[curr] = tot, ++siz[tot];
    } while(curr != x);
  }
}

inline int dp(R int x)
{
  if(siz[bel[x]] == -1) return 1;
  if(siz[bel[x]] > 1) return siz[bel[x]];
  else return dp(edges[x][0]) + 1;
}
inline void solve()
{
  using namespace IO;
  n = read();
  for (R int i = 1; i <= n; ++i) addedge(i, read());
  for (R int i = 1; i <= n; ++i) if(!dfn[i]) tarjan(i);
  for (R int i = 1; i <= n; ++i) siz[bel[i]] = (edges[i][0] == i ? -1 : siz[bel[i]]);
  for (R int i = 1; i <= n; ++i) if(siz[bel[i]] == 1) siz[bel[i]] = dp(i);
  for (R int i = 1; i <= n; ++i) printf("%d\n", siz[bel[i]] == -1 ? 1 : siz[bel[i]]);
}
}

int main()
{
  return USACO08Trick::solve(), 0;
}
```
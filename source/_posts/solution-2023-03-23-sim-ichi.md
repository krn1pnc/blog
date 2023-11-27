---
title: 2023.3.23 省选模拟赛 ichi 题解
date: 2023-03-24 22:44:24
categories: 题解
tags:
  - 数据结构
  - 线段树
  - 树状数组
  - 树套树
  - Kruskal 重构树
---

## 题面

### 题目描述

给出一棵 $n$ 个节点带边权带点权的树，请你支持两种操作：

1. $1\ x$，询问节点 $x$ 的权值．
2. $2\ v\ d\ x$，给 $x$ 的子树中到 $x$ 路径上所有边的权值均 $\ge d$ 的点的权值加 $v$．

强制在线．

### 数据范围

$1 \le n,m,d,x,y,a_i \le 10^5$．

### 时空限制

3.0 s，256 MB．

## 思路

~~没学过瓶颈路，狠狠地吃亏了．~~

考虑 $2$ 操作那个奇怪的限制，发现限制是一个类似瓶颈路的东西．于是建立 Kruskal 重构树．

我们知道，Kruskal 重构树有一个性质：对于原图中的一个节点 $u$，从其出发，只经过边权大于 $d$ 的边可以到达的节点，就是 $u$ 在重构树上满足点权 $\ge d$ 的最浅祖先子树中的点．

这样的话，$2$ 操作中 $d$ 的限制就转化成了区间限制，而又注意到将树拍成 dfn 序后，$x$ 的子树这一限制也是区间限制．这样的话，更新就是矩形加，查询就是单点查．

差分修改后查询前缀矩形和，容易使用树状数组套动态开点线段树实现．

~~你问我为什么选择树状数组？当然是因为好写．~~

时间复杂度 $O(n \log^2 n)$，空间复杂度 $O(n \log^2 n)$．

## 代码

```cpp
#include <algorithm>
#include <cstdio>
#include <numeric>
#include <vector>

int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = (c != '-') ? 1 : -1, c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return x * f;
}

constexpr int N = 1e5;
constexpr int LOGN = 20;

int n, m, type, a[N + 5];
std::vector<int> g[N + 5];
int cid, in[N + 5], out[N + 5];
void dfs(int u, int fa) {
	in[u] = ++cid;
	for (auto v : g[u]) {
		if (v == fa) continue;
		dfs(v, u);
	}
	out[u] = cid;
}

namespace segtree {
#define lch (t[p].ch[0])
#define rch (t[p].ch[1])
#define mid ((cl + cr) / 2)
struct node {
	int ch[2], s;
} t[4 * N * LOGN + 5];
int alct;
void pushup(int p) { t[p].s = t[lch].s + t[rch].s; }
void upd(int &p, int x, int d, int cl = 1, int cr = n) {
	if (!p) p = ++alct;
	if (cl == cr) return t[p].s += d, void();
	(x <= mid) ? upd(lch, x, d, cl, mid)
			   : upd(rch, x, d, mid + 1, cr);
	pushup(p);
}
int que(int p, int l, int r, int cl = 1, int cr = n) {
	if (!p) return 0;
	if (cl == l && cr == r) return t[p].s;
	if (r <= mid) return que(lch, l, r, cl, mid);
	else if (l > mid) return que(rch, l, r, mid + 1, cr);
	else return que(lch, l, mid, cl, mid)
				+ que(rch, mid + 1, r, mid + 1, cr);
}
#undef lch
#undef rch
#undef mid
} // namespace segtree

namespace fwt {
int rt[N + 5];
int lowbit(int x) { return x & (-x); }
void _upd(int x, int y, int d) {
	for (int i = x; i <= n; i += lowbit(i)) {
		segtree::upd(rt[i], y, d);
	}
}
void upd(int x1, int y1, int x2, int y2, int d) {
	_upd(x1, y1, d);
	if (y2 + 1 <= n) _upd(x1, y2 + 1, -d);
	if (x2 + 1 <= n) _upd(x2 + 1, y1, -d);
	if (x2 + 1 <= n && y2 + 1 <= n) _upd(x2 + 1, y2 + 1, d);
}
int que(int x, int y) {
	int res = 0;
	for (int i = x; i >= 1; i -= lowbit(i)) {
		res += segtree::que(rt[i], 1, y);
	}
	return res;
}
} // namespace fwt

namespace krt {
namespace dsu {
int fa[N * 2 + 5];
void init(int n) { std::iota(fa + 1, fa + 1 + n, 1); }
int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
} // namespace dsu
struct edge {
	int u, v, w;
} e[N + 5];
struct node {
	int ch[2], v;
} t[N * 2 + 5];
int fa[N * 2 + 5][LOGN + 1];
int cid, in[N * 2 + 5], out[N * 2 + 5];
void dfs(int u) {
	for (int i = 1; i <= LOGN; i++) {
		fa[u][i] = fa[fa[u][i - 1]][i - 1];
	}
	in[u] = cid + 1;
	if (!t[u].ch[0]) return out[u] = ++cid, void();
	dfs(t[u].ch[0]), dfs(t[u].ch[1]);
	out[u] = cid;
}
void build() {
	std::sort(e + 1, e + 1 + m, [](auto &a, auto &b) { return a.w > b.w; });
	dsu::init(n * 2);
	int cur = n;
	for (int i = 1; i <= n - 1; i++) {
		int fu = dsu::find(e[i].u);
		int fv = dsu::find(e[i].v);
		t[++cur] = {{fu, fv}, e[i].w};
		dsu::fa[fu] = dsu::fa[fv] = cur;
		fa[fu][0] = fa[fv][0] = cur;
	}
	dfs(cur);
}
int que(int u, int k) {
	for (int i = LOGN; i >= 0; i--) {
		if (t[fa[u][i]].v >= k) u = fa[u][i];
	}
	return u;
}
} // namespace krt

int main() {
	n = rd(), m = rd(), type = rd();
	for (int i = 1; i <= n; i++) a[i] = rd();
	for (int i = 1; i <= n - 1; i++) {
		int u = rd(), v = rd(), w = rd();
		g[u].push_back(v), g[v].push_back(u);
		krt::e[i] = {u, v, w};
	}

	dfs(1, 0), krt::build();

	int lst = 0;
	for (int i = 1; i <= m; i++) {
		int opt = rd();
		if (opt == 1) {
			int up = rd();
			int u = type ? (up + lst) % n + 1 : up;
			printf("%d\n", lst = fwt::que(in[u], krt::in[u]) + a[u]);
		} else {
			int v = rd(), d = rd(), up = rd();
			int u = type ? (up + lst) % n + 1 : up;
			int ku = krt::que(u, d);
			fwt::upd(in[u], krt::in[ku], out[u], krt::out[ku], v);
		}
	}
	return 0;
}
```

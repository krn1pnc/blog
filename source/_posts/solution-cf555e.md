---
title: Codeforces 555E Case of Computer Network 题解
date: 2023-05-19 14:36:07
categories: 题解
tags:
  - 图论
  - E-BCC
---

## 思路

首先抛出一个引理：

> E-BCC 一定可以被定向成 SCC．

证明：考虑 E-BCC 内的两个点 $u, v$，由性质可得，他们之间一定存在至少两条不重复的路，不妨将其中一条按照 $u \rightarrow v$ 的方向定向，另外一条按照 $v \rightarrow u$ 的方向定向，这样就构成了一个环，显然构成了一个 SCC．在这个 SCC 内任取两个点 $x, y$，设环上 $x$ 到 $y$ 的路径方向为 $x \rightarrow y$，在 E-BCC 内，$x$ 和 $y$ 之间一定还存在一条不同的路．将这条路按照 $y \rightarrow x$ 的方向定向，容易证明操作完仍然构成 SCC．重复进行以上操作，直到 E-BCC 内的所有点强连通，剩余的边任意定向．得证．

那么，我们将题目中给的图进行 E-BCC 缩点，如果要求 $u, v$ 在定向后联通：

1. 若 $u$ 和 $v$ 在原图中不联通，则要求一定不能被满足．
2. 若 $u$ 和 $v$ 在一个 E-BCC 内，则该要求一定可以满足．
3. 若 $u$ 和 $v$ 不在一个 E-BCC 内，由于缩点后原图变成了树，树上那条路径是唯一路径．那么设 $u, v$ 所在的 E-BCC 编号分别为 $a, b$，$a, b$ 的 LCA 为 $c$，我们给 $a$ 到 $c$ 的链上节点打上行标记，$c$ 到 $b$ 的链上节点打下行标记．如果存在节点既被打了上行标记又被打了下行标记，那么肯定存在矛盾的要求．

打标记可以通过树上差分实现．

## 代码

```cpp
#include <cstdio>
#include <vector>
#include <utility>
#include <unordered_map>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

using pii = std::pair<int, int>;

const int N = 1e6;

int n, m, q;
struct edge { int v, nxt; } g[N * 2 + 5]; int head[N + 5], ect = 1;
void add(int u, int v) {
	g[++ect] = {v, head[u]}, head[u] = ect;
	g[++ect] = {u, head[v]}, head[v] = ect;
}

int dfn[N + 5], low[N + 5], cdfn, cut[N + 5];
void tarjan(int u, int fa) {
	dfn[u] = low[u] = ++cdfn;
	for(int e = head[u]; e; e = g[e].nxt) {
		int v = g[e].v; if(v == fa) continue;
		if(!dfn[v]) {
			tarjan(v, u), low[u] = std::min(low[u], low[v]);
			if(dfn[u] < low[v]) cut[e] = cut[e ^ 1] = 1;
		} else low[u] = std::min(low[u], dfn[v]);
	}
}
int col[N + 5], cc;
void dfs(int u, int c) {
	col[u] = c;
	for(int e = head[u]; e; e = g[e].nxt) {
		int v = g[e].v; if(cut[e] || col[v]) continue;
		dfs(v, c);
	}
}

struct pii_hash {
	unsigned long long splitmix64(unsigned long long x) const {
		x += 0x9e3779b97f4a7c15;
		x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
		x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
		return x ^ (x >> 31);
	}
	unsigned long long operator()(const pii a) const {
		return splitmix64((unsigned long long)a.first << 32 | a.second);
	}
};

std::vector<int> t[N + 5];
std::unordered_map<pii, int, pii_hash> mp;

int fa[20][N + 5], dep[N + 5];
void getfa(int u, int f) {
	fa[0][u] = f, dep[u] = dep[f] + 1;
	for(int i = 1; i < 20; i++) {
		if(dep[u] < (1 << i)) continue;
		fa[i][u] = fa[i - 1][fa[i - 1][u]];
	}
	for(int v : t[u]) {
		if(v == f) continue;
		getfa(v, u);
	}
}
int lca(int u, int v) {
	if(dep[u] < dep[v]) std::swap(u, v);
	int d = dep[u] - dep[v];
	for(int i = 0; i < 20; i++) {
		if(d & (1 << i)) u = fa[i][u];
	}
	if(u == v) return u;
	for(int i = 19; i >= 0; i--) {
		if(fa[i][u] != fa[i][v]) u = fa[i][u], v = fa[i][v];
	}
	return fa[0][u];
}

namespace dsu {
	int fa[N + 5];
	void init(int n) { for(int i = 1; i <= n; i++) fa[i] = i; }
	int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
};

int vis[N + 5], up[N + 5], dn[N + 5];
void sumup(int u, int fa) {
	vis[u] = true;
	for(int v : t[u]) {
		if(v == fa) continue;
		sumup(v, u), up[u] += up[v], dn[u] += dn[v];
	}
}

int main() {
	n = rd(), m = rd(), q = rd();
	dsu::init(n);
	for(int i = 1; i <= m; i++) {
		int u = rd(), v = rd(); add(u, v);
		int fu = dsu::find(u), fv = dsu::find(v);
		if(fu != fv) dsu::fa[fu] = fv;
	}
	for(int i = 1; i <= n; i++) if(!dfn[i]) tarjan(i, i);
	for(int i = 1; i <= n; i++) if(!col[i]) dfs(i, ++cc);
	for(int u = 1; u <= n; u++) {
		for(int e = head[u]; e; e = g[e].nxt) {
			int v = g[e].v; if(col[u] == col[v] || mp.count({col[u], col[v]})) continue;
			t[col[u]].push_back(col[v]), t[col[v]].push_back(col[u]);
			mp[{col[u], col[v]}] = mp[{col[v], col[u]}] = 1;
		}
	}
	for(int i = 1; i <= cc; i++) {
		if(!dep[i]) getfa(i, 0);
	}
	for(int i = 1; i <= q; i++) {
		int u = rd(), v = rd();
		if(dsu::find(u) != dsu::find(v)) {
			puts("No");
			return 0;
		}
		int a = col[u], b = col[v];
		if(a == b) continue;
		int c = lca(a, b);
		up[a]++, up[c]--, dn[b]++, dn[c]--;
	}
	for(int i = 1; i <= cc; i++) {
		if(!vis[i]) sumup(i, i);
	}
	for(int i = 1; i <= cc; i++) {
		if(up[i] && dn[i]) {
			puts("No");
			return 0;
		}
	}
	puts("Yes");
	return 0;
}
```

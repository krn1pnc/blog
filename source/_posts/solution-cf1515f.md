---
title: Codeforces 1515F Phoenix and Earthquake 题解
date: 2023-05-22 22:44:58
categories: 题解
tags:
  - 图论
  - 生成树
---

## 思路

首先，观察到所选择的边一定构成原图中的某一生成树．

接着，我们可以得到这样的限制：若 $\sum\limits_{i = 1}^n a_i < (n - 1)x$ 或图不连通，那么一定不存在构造方案．

这样的限制乍一看非常弱，但手玩几次之后发现似乎不存在别的限制．那么我们猜测：

> 若 $\sum\limits_{i = 1}^n a_i \ge (n - 1)x$，那么原图的任意一个生成树都可以构成一个合法方案．

证明：对 $n$ 归纳．首先对于 $n = 2$ 的情况，结论显然成立．假设有 $n - 1$ 个点的情况成立，对于有 $n$ 个点的图的任一生成树，考虑它的一个叶子节点 $u$：

1. 若 $a_u \ge x$，那么我们直接选择 $u$ 和它的父亲间连的边，转化成 $n - 1$ 个点的生成树．由归纳假设可知，结论成立．

2. 若 $a_u < x$，考虑去除 $u$ 之后的生成树，不妨设 $u = n$，发现有：
   $$\sum\limits_{i = 1}^n a_i \ge (n - 1)x \iff -a_u + \sum\limits_{i = 1}^n a_i \ge -x + (n - 1)x \iff \sum_{i = 1}^{n - 1} a_i \ge (n - 2)x$$
   去除 $u$ 后的生成树满足归纳假设，结论成立．

对于构造方案，我们先随意拉出原图中的一个生成树，对其 dfs 的过程中维护两个栈 $s, t$．若子节点可以合并就之间合并，并将这条边塞到 $s$ 中，不能合并就塞到 $t$ 中．最终从栈底开始输出 $s$，再从栈顶开始输出 $t$ 即可．

## 代码

```cpp
#include <cstdio>
#include <utility>
#include <vector>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

using i64 = long long;

using pii = std::pair<int, int>;
const int N = 3e5;

int n, m;
i64 x, a[N + 5];

int fa[N + 5];
int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }

std::vector<pii> g[N + 5];
int ans[N + 5], hd, tl;
void dfs(int u, int fa) {
	for (auto [v, id] : g[u]) {
		if (v == fa) continue;
		dfs(v, u);
		if (a[u] + a[v] >= x) {
			a[u] = a[u] + a[v] - x;
			ans[hd++] = id;
		} else ans[tl--] = id;
	}
}

int main() {
	n = rd(), m = rd(), x = rd();
	for (int i = 1; i <= n; i++) a[i] = rd();

	i64 sum = 0;
	for (int i = 1; i <= n; i++) sum += a[i];
	if (sum < (n - 1) * x) return puts("NO"), 0;

	for (int i = 1; i <= n; i++) fa[i] = i;
	int ect = 0;
	for (int i = 1; i <= m; i++) {
		int u = rd(), v = rd();
		int fu = find(u), fv = find(v);
		if (fu == fv) continue;
		ect++, fa[fu] = fv;
		g[u].emplace_back(v, i);
		g[v].emplace_back(u, i);
	}
	if (ect != n - 1) return puts("NO"), 0;

	hd = 1, tl = n - 1, dfs(1, 0);

	puts("YES");
	for (int i = 1; i <= n - 1; i++) {
		printf("%d\n", ans[i]);
	}
	return 0;
}
```

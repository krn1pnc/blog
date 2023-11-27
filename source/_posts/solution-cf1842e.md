---
title: Codeforces 1842E Tenzing and Triangle 题解
date: 2023-06-26 23:37:30
categories: 题解
tags:
  - 动态规划
  - 线段树
---

## 思路

注意到选出的三角形一定没有公共部分．若存在有公共部分的两个三角形，则这两个三角形可以合并成一个大三角形，代价不增的同时可能可以覆盖到更多的点，一定更优．

于是考虑 DP．设 $f_i$ 表示考虑了 $x$ 坐标在 $[0, i]$ 内的所有点，存在一个右下角的横坐标为 $i$ 的三角形时的最小代价．枚举上一个三角形的结束位置，我们有如下转移：

$$
f_i = \min_{j < i} \{f_j + (i - j - 1)A + \mathrm{cost}\}
$$

其中 $\mathrm{cost}$ 为所有满足 $x \in (j, i], y \in [1, k - i)$ 的点的 $c$ 之和．

注意到每次 $i - 1 \rightarrow i$ 的过程中，只有满足 $x = i$ 或 $y = k - i$ 的点会对一些 $j$ 的 $\mathrm{cost}$ 造成影响，故考虑直接维护 $\min$ 里面的东西．

具体而言，在 $i - 1 \rightarrow i$ 的过程中，我们会有一些形如 $(p, k - i)$ 的点，这些点的 $c$ 需要从 $p$ 之前的 $\mathrm{cost}$ 中去掉；我们还会有一些形如 $(i, \dots)$ 的点，这些点的 $c$ 需要加入 $i$ 之前的 $\mathrm{cost}$ 中．这些操作对 $\min$ 里面东西的影响都形如前缀加，而 $(i - j - 1)A$ 这一项的贡献也是前缀加．用一个支持前缀加前缀 $\min$ 的数据结构维护即可．

注意 $-1$ 处的东西是有用的，表示所有点都未被考虑的状态，需要维护．

## 代码

使用线段树实现．

```cpp
#include <algorithm>
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

using pii = std::pair<int, int>;
using i64 = long long;
const int N = 2e5;

int n, k, A;

#define lch (p * 2)
#define rch (p * 2 + 1)
#define mid ((cl + cr) >> 1)
struct node {
	i64 mn, tg;
	void upd(i64 d) {
		mn += d, tg += d;
	}
} t[N * 4 + 5];
void pushup(int p) { t[p].mn = std::min(t[lch].mn, t[rch].mn); }
void pushdown(int p) {
	if (!t[p].tg) return;
	t[lch].upd(t[p].tg), t[rch].upd(t[p].tg);
	t[p].tg = 0;
}
void upd(int l, int r, i64 d, int p = 1, int cl = -1, int cr = k) {
	if (cl == l && cr == r) return t[p].upd(d);
	pushdown(p);
	if (r <= mid) upd(l, r, d, lch, cl, mid);
	else if (l > mid) upd(l, r, d, rch, mid + 1, cr);
	else upd(l, mid, d, lch, cl, mid), upd(mid + 1, r, d, rch, mid + 1, cr);
	pushup(p);
}
i64 que(int l, int r, int p = 1, int cl = -1, int cr = k) {
	if (cl == l && cr == r) return t[p].mn;
	pushdown(p);
	if (r <= mid) return que(l, r, lch, cl, mid);
	else if (l > mid) return que(l, r, rch, mid + 1, cr);
	else return std::min(que(l, mid, lch, cl, mid), que(mid + 1, r, rch, mid + 1, cr));
}
#undef lch
#undef rch
#undef mid

std::vector<pii> a[N + 5];
i64 ct[N + 5], f[N + 5];

int main() {
	n = rd(), k = rd(), A = rd();
	for (int i = 1; i <= n; i++) {
		int x = rd(), y = rd(), c = rd();
		a[y].emplace_back(x, c);
		if (x + y != k) ct[x] += c;
	}
	for (int i = 0; i <= k; i++) {
		for (auto [x, c] : a[k - i]) upd(-1, x - 1, -c);
		upd(-1, i - 1, ct[i]);
		if (i != 0) upd(-1, i - 2, A);
		f[i] = que(-1, i - 1), upd(i, i, f[i]);
	}
	printf("%lld\n", f[k]);
	return 0;
}
```

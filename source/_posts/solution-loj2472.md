---
title: LOJ 2472 「九省联考 2018」IIIDX 题解
date: 2023-06-09 16:43:19
categories: 题解
tags:
  - 贪心
  - 线段树
  - 二分
---

## 思路

容易观察到难度限制就是让我们构造小根堆，于是 $d_i$ 互不相同时贪心地给编号较小的子树分配较大的权值即可．记 $\mathrm{size}_u$ 代表 $u$ 子树的大小．

$d_i$ 中存在相同元素时，上述贪心策略寄了．为什么？考虑一个点 $u$，在我们分配完成后，若我们能够将分配给 $u$ 的子树中某个节点 $v$ 的权值与分配给 $u$ 的某个编号大于 $u$ 的兄弟节点 $x$ 的权值交换，我们就能够通过这样的调整来增大答案的字典序．当 $d_i$ 互不相同时，这样的 $u$ 显然不存在；当 $d_i$ 中存在相同元素时，$u$ 的子树中可以填和自己相同的值，而按照贪心策略则会尽量填大的值，这显然是不优的．

要求字典序最大，我们应当考虑按照 $1 \sim n$ 的顺序按位确定权值．而我们又发现，如果我们已经确定完 $1 \sim u - 1$ 所填的权值，决策节点 $u$ 的权值时，若这个值选得太大，则子树中没有足够多的权值去填充．这说明节点 $u$ 能填的权值存在一个上界，我们尝试去二分这个上界，贪心地选择它作为答案．

问题转化为判定性问题，我们想要判断的是，如果已经确定了 $1 \sim u - 1$ 的权值，节点 $u$ 上填 $x$ 这个值后是否存在一个合法的填数方案．由于 $1 \sim u - 1$ 的权值已经确定，那么直接与未确定的点相连的节点对其子树中所填的值都有限制．于是我们维护 $\mathrm{ct}_i$ 表示 $i$ 这个值还剩多少个，$\mathrm{book}_i$ 表示当前填 $i$ 这个值的所有节点子树中需要多少个大于等于 $i$ 的值．当 $u$ 填 $x$ 时，$\mathrm{ct}_x$ 会减去 $1$，$\mathrm{book}_x$ 会减去 $\mathrm{size}_u - 1$．由于我们已经确定了的点在树上构成一个联通块，子树限制必然不交，也就是说 $\sum_{j \ge x} \mathrm{book}_j$ 就是当前所需 $\ge x$ 权值的个数．那么判断存在合法方案时，直接假设 $u$ 填了 $x$，然后只需要判断 $\mathrm{ct}$ 的后缀和是否处处不小于 $\mathrm{book}$ 的后缀和即可．

暴力做的时间复杂度是 $O(n^2 \log n)$ 的，可以拿到和贪心一样的分数．

于是考虑如何优化这个二分过程，如果我们记 $s_i = \sum_{j \ge i} \mathrm{ct}_i - \mathrm{book}_i$，每次确定 $u$ 的值时对 $s$ 的影响形如前缀减 $\mathrm{size}_u$，而二分实际上时找到最大的 $x$，使得 $\forall i \le x, s_i - \mathrm{size}_u \ge 0$．于是使用线段树维护 $s$ 的最小值，支持区间加法即可．

注意决策到 $u$ 时需先撤销先前决策 $\lfloor u / k \rfloor$ 时 $u$ 子树对 $\mathrm{book}$ 的影响．

二分可以在线段树上进行，时间复杂度 $O(n \log n)$．

## 代码

实现细节有点多．

```cpp
#include <algorithm>
#include <cstdio>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

const int N = 5e5;

int n, a[N + 5];
double k;
int b[N + 5], bct;

int fa[N + 5], siz[N + 5];
int val[N + 5], ct[N + 5], bk[N + 5];

#define lch (p * 2)
#define rch (p * 2 + 1)
#define mid ((cl + cr) / 2)
struct node {
	int mn, tg;
	void upd(int d) {
		mn += d, tg += d;
	}
} t[N * 4 + 5];
void pushup(int p) { t[p].mn = std::min(t[lch].mn, t[rch].mn); }
void pushdown(int p) {
	if (!t[p].tg) return;
	t[lch].upd(t[p].tg);
	t[rch].upd(t[p].tg);
	t[p].tg = 0;
}
void upd(int l, int r, int d, int p = 1, int cl = 1, int cr = bct) {
	if (cl == l && cr == r) return t[p].upd(d);
	pushdown(p);
	if (r <= mid) upd(l, r, d, lch, cl, mid);
	else if (l > mid) upd(l, r, d, rch, mid + 1, cr);
	else upd(l, mid, d, lch, cl, mid), upd(mid + 1, r, d, rch, mid + 1, cr);
	pushup(p);
}
int getpos(int k, int p = 1, int cl = 1, int cr = bct) {
	if (cl == cr) return cl - (t[p].mn < k);
	pushdown(p);
	if (t[lch].mn >= k) return getpos(k, rch, mid + 1, cr);
	else return getpos(k, lch, cl, mid);
}
#undef lch
#undef rch
#undef mid

int main() {
	n = rd(), scanf("%lf", &k);
	for (int i = 1; i <= n; i++) a[i] = b[i] = rd();
	std::sort(b + 1, b + 1 + n), bct = std::unique(b + 1, b + 1 + n) - b - 1;
	for (int i = 1; i <= n; i++) a[i] = std::lower_bound(b + 1, b + 1 + bct, a[i]) - b;

	for (int i = 1; i <= n; i++) upd(1, a[i], 1);
	for (int i = 1; i <= n; i++) siz[i] = 1, fa[i] = i / k;
	for (int i = n; i >= 1; i--) siz[fa[i]] += siz[i];

	val[0] = 1, upd(1, 1, -n);
	for (int u = 1; u <= n; u++) {
		upd(1, val[fa[u]], siz[u]);
		int ans = getpos(siz[u]);
		val[u] = ans;
		upd(1, ans, -siz[u]);
	}

	for (int i = 1; i <= n; i++) {
		printf("%d%c", b[val[i]], " \n"[i == n]);
	}
	return 0;
}
```

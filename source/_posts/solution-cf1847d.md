---
title: Codeforces 1847D Professor Higashikata 题解
date: 2023-07-07 21:51:02
categories: 题解
tags:
  - 线段树
  - 贪心
---

## 思路

先考虑区间不交的情况．

容易观察到，若给出的区间不交，操作后拼接出的串的一定形如若干个 $1$ 拼接上若干个 $0$．

考虑如何计算操作数．由于区间不交，设区间并的大小为 $k$，那么我们可以构造一个置换 $p$，使得 $p$ 作用在原序列上后，新序列的前 $k$ 位恰好对应拼接出的串．再令当前序列中 $1$ 的个数为 $w$，由于新序列中长度为 $\min\{k, w\}$ 的前缀在操作后应当全为 $1$，那么操作次数就是这个前缀中 $0$ 的个数，容易维护．

构造 $p$ 可以依次拼接区间．具体地，我们维护变量 $\mathrm{tp}$，初始为 $0$．然后按照顺序遍历区间，每到一个位置 $i$，先令 $\mathrm{tp}$ 自增，然后将 $p_i$ 赋值为 $\mathrm{tp}$．最后仍未确定的元素可以瞎填，保证 $p$ 仍然是一个置换即可．

考虑将这个做法扩展到区间有交的情况．两区间有交时，需优先保证编号小的区间被填 $1$．那么我们在构造 $p$ 的方式上动一点手脚，不对已经确定的元素赋值，这样构造出的置换作用在原序列上后，新序列的前 $k$ 位恰好对应将编号较大的区间中与编号较小的区间的交去掉后拼接出的串．为了使高位尽可能填 $1$，操作后的新序列长度为 $\min\{k, w\}$ 的前缀也应当全为 $1$．维护答案的方式不变．

这时如果我们暴力构造 $p$，时间复杂度会退化到 $O(n^2)$，所以我们应当只填没有填过的位置．这里有很多实现方式，我使用了一棵线段树．具体地，维护区间内第一个未被填过和被填过的位置，每次找到 $[l, r]$ 内第一个未被填过的位置，记作 $a$，再找到 $[a, r]$ 内第一个被填过的位置，记作 $b$，$[a, b)$ 就是区间 $[l, r]$ 内第一个未被填过的区间，暴力填即可．不断重复这一次操作，时间复杂度 $O(n \log n)$．

## 代码

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

const int N = 2e5;

int n, m, q, a[N + 5];

#define lch (p * 2)
#define rch (p * 2 + 1)
#define mid ((t[p].l + t[p].r) / 2)
struct dat {
	int f0, f1;
};
dat operator+(dat a, dat b) {
	dat res;
	if (a.f0 == -1) {
		if (b.f0 == -1) res.f0 = -1;
		else res.f0 = b.f0;
	} else res.f0 = a.f0;
	if (a.f1 == -1) {
		if (b.f1 == -1) res.f1 = -1;
		else res.f1 = b.f1;
	} else res.f1 = a.f1;
	return res;
}
struct node {
	int l, r;
	dat v;
	int tg;
	void upd() {
		v = {-1, l};
		tg = 1;
	}
} t[N * 4 + 5];
void pushup(int p) { t[p].v = t[lch].v + t[rch].v; }
void pushdown(int p) {
	if (!t[p].tg) return;
	t[lch].upd(), t[rch].upd();
	t[p].tg = 0;
}
void build(int p = 1, int cl = 1, int cr = n) {
	t[p].l = cl, t[p].r = cr;
	if (cl == cr) return t[p].v = {cl, -1}, void();
	build(lch, cl, mid), build(rch, mid + 1, cr);
	pushup(p);
}
void upd(int l, int r, int p = 1) {
	if (l > r) return;
	if (t[p].l == l && t[p].r == r) return t[p].upd();
	pushdown(p);
	if (r <= mid) upd(l, r, lch);
	else if (l > mid) upd(l, r, rch);
	else upd(l, mid, lch), upd(mid + 1, r, rch);
	pushup(p);
}
dat que(int l, int r, int p = 1) {
	if (l > r) return {-1, -1};
	if (t[p].l == l && t[p].r == r) return t[p].v;
	pushdown(p);
	if (r <= mid) return que(l, r, lch);
	else if (l > mid) return que(l, r, rch);
	else return que(l, mid, lch) + que(mid + 1, r, rch);
}
#undef lch
#undef rch
#undef mid

namespace fwt {
	int t[N + 5];
	int lowbit(int x) { return x & (-x); }
	void upd(int x, int d) {
		for (int i = x; i <= n; i += lowbit(i)) {
			t[i] += d;
		}
	}
	int que(int x) {
		int res = 0;
		for (int i = x; i >= 1; i -= lowbit(i)) {
			res += t[i];
		}
		return res;
	}
};

int p[N + 5], b[N + 5];

char s[N + 5];
int main() {
	n = rd(), m = rd(), q = rd(), scanf("%s", s + 1);
	for (int i = 1; i <= n; i++) a[i] = s[i] - '0';

	build(); int tp = 0;
	for (int _ = 1; _ <= m; _++) {
		int l = rd(), r = rd(), i = l, j = r;
		while (i <= j) {
			int a = que(i, j).f0; if (a == -1) a = j + 1;
			int b = que(a, j).f1; if (b == -1) b = j + 1;
			for (int k = a; k <= b - 1; k++) p[k] = ++tp;
			i = b + 1;
		}
		upd(l, r);
	}
	int lim = tp;
	for (int i = 1; i <= n; i++) {
		if (!p[i]) p[i] = ++tp;
	}

	for (int i = 1; i <= n; i++) b[p[i]] = a[i];
	for (int i = 1; i <= n; i++) fwt::upd(i, b[i]);

	for (int i = 1; i <= q; i++) {
		int pos = p[rd()]; b[pos] ^= 1;
		b[pos] ? fwt::upd(pos, 1) : fwt::upd(pos, -1);
		int tot = std::min(lim, fwt::que(n));
		int x = tot - fwt::que(tot);
		printf("%d\n", x);
	}
	return 0;
}
```

---
title: 洛谷 3732 [HAOI2017] 供给侧改革
date: 2023-05-27 23:24:34
categories: 题解
tags:
  - Trie
  - 线段树
  - 随机
  - 扫描线
---

## 思路

首先，由于数据随机，不妨猜测 LCP 的最大长度不超过某个小常数 $K$．记 $S_i$ 表示由 $i$ 开始的后缀的长度为 $K$ 的前缀．

询问比较诡异，猜测难以高效地在线解决，于是考虑离线．将询问挂在右端点上，扫描线解决每个询问．对于当前扫描到的右端点 $r$，维护 $X_i = \operatorname{data}(i, r)$，询问就是对 $X_i$ 区间求和，丢到线段树上即可．

考虑移动右端点，加入 $S_r$ 时对 $X_i$ 的影响．由于 $K$ 较小，我们不妨暴力修改．具体而言，我们枚举 $S_r$ 的前缀 $T$，钦定 $T$ 为 LCP，然后找到最大的 $p$，使得 $S_p$ 含有 $T$ 这个前缀．对于 $X_j(1 \le j \le p)$，$T$ 作为 LCP 都能对其造成贡献，$X_j$ 需要对 $|T|$ 取 $\max$．

如何找到 $p$？考虑 Trie 的性质，对于一个串 $S$，它的一个前缀在 Trie 上对应的节点在 $S$ 对应的节点的祖先链上．那么我们维护一棵插入了 $S_i(1 \le i < r)$ 的 Trie，对 Trie 上每个节点 $u$ 维护 $\mathrm{mx}_u$，表示 $u$ 对应的串最后一次作为前缀出现是在 $S_{\mathrm{mx}_u}$ 中．那么对于 $S_r$ 的一个前缀，设它在 Trie 上对应的节点为 $u$，我们要找的 $p$ 就是 $\mathrm{mx}_u$．

对于区间取 $\max$ 操作，乍一看需要一棵维护区间历史最值的线段树．然而我们发现，对于 $u < v$，$X_u$ 中会包含 $X_v$ 的贡献，即 $X_i$ 单调不增，所谓的取 $\max$ 操作其实是对某个右端点为 $p$ 的区间的覆盖，线段树维护区间最小值就可以二分出区间左端点．这样就只需要一棵支持区间覆盖，求区间最小值和区间和的线段树．

在 Trie 上枚举前缀，复杂度省掉一个 $K$．

时间复杂度 $O(qK\log n)$，$K$ 取 $32$ 即可通过本题．

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

using pii = std::pair<int, int>;

const int N = 1e5;
const int K = 32;

int n, q;
char a[N + 5];

std::vector<pii> qu[N + 5];
int ans[N + 5];

int ch[N * K + 5][2], dep[N * K + 5], mx[N * K + 5], alct;
void ins(char *s, int len, int id) {
	int p = 0;
	for (int i = 1; i <= len; i++) {
		int d = s[i] - '0';
		if (!ch[p][d]) ch[p][d] = ++alct;
		p = ch[p][d], mx[p] = id, dep[p] = i;
	}
}

#define lch (p * 2)
#define rch (p * 2 + 1)
#define mid ((t[p].l + t[p].r) / 2)
struct node {
	int l, r;
	int sum, mn;
	int tg;
	void upd(int d) {
		sum = d * (r - l + 1);
		mn = d, tg = d;
	}
} t[N * 4 + 5];
void pushup(int p) {
	t[p].sum = t[lch].sum + t[rch].sum;
	t[p].mn = std::min(t[lch].mn, t[rch].mn);
}
void pushdown(int p) {
	if (!t[p].tg) return;
	t[lch].upd(t[p].tg);
	t[rch].upd(t[p].tg);
	t[p].tg = 0;
}
void build(int p = 1, int cl = 1, int cr = n) {
	t[p].l = cl, t[p].r = cr;
	if (cl == cr) return;
	build(lch, cl, mid), build(rch, mid + 1, cr);
}
void upd(int l, int r, int d, int p = 1) {
	if (t[p].l == l && t[p].r == r) return t[p].upd(d);
	pushdown(p);
	if (r <= mid) upd(l, r, d, lch);
	else if (l > mid) upd(l, r, d, rch);
	else upd(l, mid, d, lch), upd(mid + 1, r, d, rch);
	pushup(p);
}
int que(int l, int r, int p = 1) {
	if (t[p].l == l && t[p].r == r) return t[p].sum;
	pushdown(p);
	if (r <= mid) return que(l, r, lch);
	else if (l > mid) return que(l, r, rch);
	else return que(l, mid, lch) + que(mid + 1, r, rch);
}
int getpos(int k, int p = 1) {
	if (t[p].l == t[p].r) return t[p].mn < k ? t[p].l : 0;
	pushdown(p);
	return (t[lch].mn < k) ? getpos(k, lch) : getpos(k, rch);
}
#undef lch
#undef rch
#undef mid

void solve(char *s, int len, int id) {
	int p = 0;
	for (int i = 1; i <= len; i++) {
		int d = s[i] - '0';
		if (!ch[p][d]) break;
		p = ch[p][d];
		int pos = getpos(dep[p]);
		if (pos && pos <= mx[p]) {
			upd(pos, mx[p], dep[p]);
		}
	}
	ins(s, len, id);
}

int main() {
	n = rd(), q = rd();
	scanf("%s", a + 1);
	for (int i = 1; i <= q; i++) {
		int l = rd(), r = rd();
		qu[r].emplace_back(l, i);
	}

	build();
	for (int i = 1; i <= n; i++) {
		solve(a + i - 1, std::min(n - i + 1, K), i);
		for (auto [l, id] : qu[i]) ans[id] = que(l, i);
	}

	for (int i = 1; i <= q; i++) printf("%d\n", ans[i]);
	return 0;
}
```

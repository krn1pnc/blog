---
title: 2023.3.25 省选模拟赛 Arithmetic 题解
date: 2023-03-28 21:47:21
categories: 题解
tags:
  - 数据结构
  - 扫描线
  - 线段树
  - 矩阵
---

## 题面

### 题目描述

对于一个整数序列 $A$，你可以进行两种操作：

- 在序列任意位置插入一个任意值的元素，花费为 $1$；
- 交换序列中任意两个元素的值，花费为 $0$．

对于给定的常数 $d$，定义 $f(A)$ 为将 $A$ 变为一个公差为 $d$ 的等差数列所需的最小花费．特别地，若无论怎样操作都无法变为一个公差为 $d$ 的等差序列，定义 $f(A) = 0$．

现在给出一个长为 $n$ 的序列 $S$，$q$ 次询问，每次给出 $l,r$，你需要回答：
$$
\sum_{i = l}^{r}\sum_{j = i}^{r} f(S[i \dots j])
$$
其中 $S[i \dots j]$ 表示 $S$ 的第 $i$ 个元素到第 $j$ 个元素构成的子串（从 $1$ 开始标号）．

### 数据范围

$0 \le n,q \le 3 \times 10^5$，$0 \le S_i,d \le 10^7$．

### 时空限制

2.0 s，512 MB．

## 思路

~~考场上没卡常，直接挂成暴力．~~

先考虑对于一个确定的序列，如何计算 $f$．

我们称一个序列合法，当且仅当：

1. 序列内没有重复的数．
2. 序列内的数模 $d$ 意义下同余．

容易看出，不合法序列的 $f$ 值一定为 $0$．

对于一个合法序列 $A$，记 $p = \max\limits_i \lfloor A_i / d \rfloor$，$q = \min\limits_i \lfloor A_i / d \rfloor$，有 $f(A) = p - q + 1 - |A|$，其中 $|A|$ 代表 $A$ 的长度．

接着我们考虑区间询问，发现这是对区间子区间求和，尝试使用扫描线来解决．

考虑对于一个确定的 $r$，如何对任意的 $l$ 计算区间 $[l,r]$ 的答案．

记 $X_i = \sum\limits_{i \le j \le r} f(S[i \dots j])$．对于询问区间 $[l,r]$，答案显然为 $\sum\limits_{l \le i \le r} X_i$．

再记 $p_{l,r} = \max\limits_{l \le i \le r} \lfloor S_i / d \rfloor$，$q_{l,r} = \min\limits_{l \le i \le r} \lfloor S_i / d \rfloor$，考虑当前扫描到 $r - 1$，加入 $S_r$ 对于 $X$ 的影响，设 $k$ 为最小满足 $S[k \dots r]$ 是合法序列的位置，这个可以通过记录每个数的最后出现位置和模 $d$ 不同余的数的最后位置确定（当然如果你觉得不好理解可以选择糊一棵动态开点权值线段树，不过貌似空间很寄）．那么对于所有 $X_i, k \le i \le r$，都会加上 $p_{i,r} - q_{i,r} + 1 - (r - i + 1) = p_{i,r} - q_{i,r} + i - r$．

注意到有两个形如后缀最值的项，插入时对后缀最值的影响形如区间覆盖．考虑在线段树上记录这些值，扫描过程中维护两个单调栈，用于求出后缀最值修改的左端点．注意到式子长得很好看，允许我们写成矩阵的形式糊上去．

具体地，扫描 $r$ 时，对于每个位置 $i$，我们在线段树上维护向量 $\begin{bmatrix} X & p & q & i & 1 \end{bmatrix}$ 的区间和，其中 $X$、$p$、$q$ 分别代表 $X_i$、$p_{i,r}$、$q_{i,r}$．区间 $p$ 覆盖 $x$、区间 $q$ 覆盖 $x$、对于右端点为 $r$ 的区间更新答案的修改矩阵分别如下：
$$
\begin{bmatrix}
  1 & 0 & 0 & 0 & 0 \\
  0 & 0 & 0 & 0 & 0 \\
  0 & 0 & 1 & 0 & 0 \\
  0 & 0 & 0 & 1 & 0 \\
  0 & x & 0 & 0 & 1 \\
\end{bmatrix} ,
\begin{bmatrix}
  1 & 0 & 0 & 0 & 0 \\
  0 & 1 & 0 & 0 & 0 \\
  0 & 0 & 0 & 0 & 0 \\
  0 & 0 & 0 & 1 & 0 \\
  0 & 0 & x & 0 & 1 \\
\end{bmatrix} ,
\begin{bmatrix}
  1 & 0 & 0 & 0 & 0 \\
  1 & 1 & 0 & 0 & 0 \\
  -1 & 0 & 1 & 0 & 0 \\
  1 & 0 & 0 & 1 & 0 \\
  -r & 0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

当然，如果你~~和我一样~~直接糊矩阵上去，会自带一个 $125$ 的常数，难以通过本题．使用 [拆矩阵的技巧](/posts/trick-matrix-tag-optimize) 即可跑过，~~比 std 快到不知道哪里去了~~．

这种求和貌似有个名字叫区间历史和？不是很懂啊．

## 代码

```cpp
#include <algorithm>
#include <cstdio>
#include <utility>
#include <vector>

int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = (c != '-') ? 1 : -1, c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return x * f;
}

using i64 = long long;
constexpr int N = 3e5;
constexpr int V = 1e7;

int n, d, m, a[N + 5];
std::vector<std::pair<int, int>> q[N + 5];

namespace segtree {
struct mat {
	i64 c21, c22, c31, c33, c41, c51, c52, c53;

	friend mat operator*(const mat &A, const mat &B) {
		mat R{};
		R.c21 = A.c21 + A.c22 * B.c21;
		R.c22 = A.c22 * B.c22;
		R.c31 = A.c31 + A.c33 * B.c31;
		R.c33 = A.c33 * B.c33;
		R.c41 = A.c41 + B.c41;
		R.c51 = A.c51 + A.c52 * B.c21 + A.c53 * B.c31 + B.c51;
		R.c52 = A.c52 * B.c22 + B.c52;
		R.c53 = A.c53 * B.c33 + B.c53;
		return R;
	}
	static mat I() {
		mat R{};
		R.c22 = R.c33 = 1;
		return R;
	}
	static mat A(int a) {
		mat R{};
		R.c33 = 1, R.c52 = a;
		return R;
	}
	static mat B(int a) {
		mat R{};
		R.c22 = 1, R.c53 = a;
		return R;
	}
	static mat C(int a) {
		mat R{};
		R.c22 = R.c33 = 1;
		R.c21 = 1, R.c31 = -1, R.c41 = 1, R.c51 = -a;
		return R;
	}
};
struct vec {
	i64 c1, c2, c3, c4, c5;

	friend vec operator+(const vec &a, const vec &b) {
		vec r{};
		r.c1 = a.c1 + b.c1;
		r.c2 = a.c2 + b.c2;
		r.c3 = a.c3 + b.c3;
		r.c4 = a.c4 + b.c4;
		r.c5 = a.c5 + b.c5;
		return r;
	}
	friend vec operator*(const vec &a, const mat &B) {
		vec r{};
		r.c1 = a.c1 + a.c2 * B.c21 + a.c3 * B.c31 + a.c4 * B.c41 + a.c5 * B.c51;
		r.c2 = a.c2 * B.c22 + a.c5 * B.c52;
		r.c3 = a.c3 * B.c33 + a.c5 * B.c53;
		r.c4 = a.c4;
		r.c5 = a.c5;
		return r;
	}
};

#define lch (p * 2)
#define rch (p * 2 + 1)
#define mid ((cl + cr) / 2)
struct node {
	vec v;
	mat T;
	bool t;
	void upd(mat D) {
		v = v * D;
		T = T * D;
		t = true;
	}
} t[N * 4 + 5];
void pushup(int p) { t[p].v = t[lch].v + t[rch].v; }
void pushdown(int p) {
	if (!t[p].t) return;
	t[lch].upd(t[p].T), t[rch].upd(t[p].T);
	t[p].T = mat::I(), t[p].t = false;
}
void build(int p = 1, int cl = 1, int cr = n) {
	t[p].T = mat::I();
	if (cl == cr) return t[p].v = {0, 0, 0, cl, 1}, void();
	build(lch, cl, mid), build(rch, mid + 1, cr);
	pushup(p);
}
void upd(int l, int r, const mat &D, int p = 1, int cl = 1, int cr = n) {
	if (cl == l && cr == r) return t[p].upd(D);
	pushdown(p);
	if (r <= mid) upd(l, r, D, lch, cl, mid);
	else if (l > mid) upd(l, r, D, rch, mid + 1, cr);
	else upd(l, mid, D, lch, cl, mid),
		 upd(mid + 1, r, D, rch, mid + 1, cr);
	pushup(p);
}
i64 que(int l, int r, int p = 1, int cl = 1, int cr = n) {
	if (cl == l && cr == r) return t[p].v.c1;
	pushdown(p);
	if (r <= mid) return que(l, r, lch, cl, mid);
	else if (l > mid) return que(l, r, rch, mid + 1, cr);
	else return que(l, mid, lch, cl, mid)
				+ que(mid + 1, r, rch, mid + 1, cr);
}
#undef lch
#undef rch
#undef mid
}; // namespace segtree

using segtree::mat;

int apr[V + 5];
int mx[2], mxp[2];

int sa[N + 5], ta;
int sb[N + 5], tb;

i64 ans[N + 5];

int main() {
	n = rd(), d = rd(), m = rd();
	if (n == 0 && m == 0) return 0;
	for (int i = 1; i <= n; i++) a[i] = rd();
	for (int i = 1, l, r; i <= m; i++) {
		l = rd(), r = rd(), q[r].emplace_back(l, i);
	}

	segtree::build();
	int pmx = 0;
	for (int r = 1; r <= n; r++) {
		while (ta && a[sa[ta]] / d < a[r] / d) ta--;
		segtree::upd(sa[ta] + 1, r, mat::A(a[r] / d));
		sa[++ta] = r;
		while (tb && a[sb[tb]] / d > a[r] / d) tb--;
		segtree::upd(sb[tb] + 1, r, mat::B(a[r] / d));
		sb[++tb] = r;

		pmx = std::max(pmx, apr[a[r]]);

		if (mx[0] == a[r] % d) mxp[0] = r;
		else mxp[1] = mxp[0], mx[1] = mx[0], mx[0] = a[r] % d, mxp[0] = r;
		apr[a[r]] = r;

		int p = std::max(mxp[1], pmx);

		segtree::upd(p + 1, r, mat::C(r));

		for (auto [l, id] : q[r]) ans[id] = segtree::que(l, r);
	}

	for (int i = 1; i <= m; i++) printf("%lld\n", ans[i]);
	return 0;
}
```

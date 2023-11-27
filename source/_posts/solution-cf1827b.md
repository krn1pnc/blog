---
title: Codeforces 1827B Range Sorting 题解
date: 2023-09-22 20:20:54
categories: 题解
tags:
  - 单调栈
  - 扫描线
---

## 思路

考虑如何计算一个长度为 $n$ 的序列的代价．

显然可以直接排序，所以答案有上界 $n - 1$．由于一段区间的代价是 $r - l$，即区间内形如 $i \sim i + 1$ 的间断数．那么我们需要拆出尽量多的线段进行操作，代价就是必须保留的间断数．

考虑一个间断 $i \sim i + 1$ 不能被舍弃的条件．若 $\max\limits_{j \le i} a_j > \min\limits_{j > i} a_j$，那么 $\argmax\limits_{j \le i} a_j$ 和 $\argmin\limits_{j > i} a_j$ 必须需要交换，而舍弃这个间断会导致 $[1, i]$ 和 $[i + 1, n]$ 中的元素无法交换，所以这个间断必须保留．

考虑对所有子区间计算．由于有上述分析，将贡献拆到间断上，即对每一个间断 $i \sim i + 1$，计算有多少 $[l, r]$，满足 $\max\limits_{l \le j \le i} a_j > \min\limits_{i < j \le r} a_j$．扫描 $i$，由于左边是一段后缀的 $\max$，考虑使用单调栈维护，对于单调栈中相邻的两个元素 $p, q$，开始于区间 $[p + 1, q]$ 内的后缀的 $\max$ 都是 $a_p$，我们所要统计的就是有多少个 $r$ 满足 $a_p > \min\limits_{i < j \le r} a_j$，这个倒着再扫一遍就行．

注意维护单调栈时，弹完栈后，设栈顶为 $p$，那么 $l \in [1, p]$ 和 $r \in [i, n]$ 都能产生贡献，记得加上去．

## 代码

```cpp
#include <cstdio>
#include <algorithm>
#include <utility>
#include <vector>
#define debug(...) fprintf(stderr, __VA_ARGS__)

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

int n, a[N + 5];
int b[N + 5];

namespace fwt {
	int t[N + 5];
	inline int lowbit(int x) { return x & (-x); }
	void init() { for(int i = 1; i <= n; i++) t[i] = 1e9; }
	void upd(int x, int v) {
		for(int i = x; i <= n; i += lowbit(i)) {
			t[i] = std::min(t[i], v);
		}
	}
	int que(int x) {
		int res = 1e9;
		for(int i = x; i >= 1; i -= lowbit(i)) {
			res = std::min(res, t[i]);
		}
		return res;
	}
};

std::vector<pii> q[N + 5];
void solve() {
	n = rd(); for(int i = 1; i <= n; i++) a[i] = b[i] = rd();
	std::sort(b + 1, b + 1 + n); int bct = std::unique(b + 1, b + 1 + n) - b - 1;
	for(int i = 1; i <= n; i++) a[i] = std::lower_bound(b + 1, b + 1 + bct, a[i]) - b;

	i64 ans = 0;
	static int st[N + 5]; int tp = 0;
	for(int i = 1; i <= n; i++) {
		while(tp && a[st[tp]] <= a[i]) {
			q[i + 1].emplace_back(a[st[tp]], st[tp] - st[tp - 1]);
			tp--;
		}
		ans += 1ll * st[tp] * (n - i + 1);
		st[++tp] = i;
	}
	fwt::init();
	for(int i = n; i >= 1; i--) {
		fwt::upd(a[i], i);
		for(auto [x, coff] : q[i]) {
			ans += 1ll * coff * (n - std::min(n + 1, fwt::que(x)) + 1);
		}
	}
	printf("%lld\n", ans);

	for(int i = 1; i <= n + 1; i++) q[i].clear();
}

int main() {
	int T = rd();
	while(T--) solve();
	return 0;
}
```

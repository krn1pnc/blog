---
title: Codeforces 650D Zip-line 题解
date: 2023-05-29 19:05:17
categories: 题解
tags:
  - 动态规划
  - LIS
  - 树状数组
---

LIS，太奇妙．

## 思路

考虑替换序列中的一个元素对 LIS 的影响．设原序列的 LIS 长度为 $k$，由于我们每次只会替换一个元素，所以对于替换后的序列，LIS 长度只有 $3$ 种取值：$k - 1$，$k$，$k + 1$．

考虑对于询问 $i$，替换后序列的 LIS 的长度何时会取到这些值．假设我们已经计算出了包含 $b_i$ 的 LIS 的长度 $t$：

1. 若 $t \ge k$，那么答案显然为 $\max\{t, k\}$．
2. 若 $t < k$，我们发现答案为 $t$ 的充要条件是替换 $h_{a_i}$ 这一操作破坏了原序列中所有 LIS，否则答案仍然为 $k$．这一条件等价于对于序列中所有可能的 LIS，出现在 LIS 的某一位上的元素只有 $h_{a_i}$．

考虑如何计算 $t$．我们可以预处理出 $f_i$、$g_i$，分别表示以 $h_i$ 结束的 LIS 长度和以 $h_i$ 开头的 LIS 长度．考虑用 $b_i$ 连接两个上升序列，得到 LIS，我们有 $t = \max\limits_{j < a_i, h_j < b_i} f_j + 1 + \max\limits_{a_i < j, b_i < h_j} g_j$，这个 $\max$ 是二维数点的形式，扫描线一下就可以求出．

考虑如何判定 2 情况中答案是否为 $t$．我们可以计算出原序列的 LIS 中，$i$ 位置上能够填的元素数量，若这个数量为 $1$，那么替换 $i$ 位置上的元素就会破坏所有的 LIS．如何计算出这个数量？枚举位置 $i$，若 $f_i + g_i - 1 = k$，说明 $h_i$ 至少出现在了一个 LIS 中，且出现位置为 $f_i$，用一个桶计数即可．

## 代码

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
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
const int N = 1e6;

int n, m, a[N + 5];
int b[N + 5], bct;

std::vector<pii> q[N + 5];
int ans[N + 5];

namespace fwt {
	int t[N + 5];
	inline int lowbit(int x) { return x & (-x); }
	inline void upd(int x, int d) {
		for (int i = x; i <= bct; i += lowbit(i)) {
			t[i] = std::max(t[i], d);
		}
	}
	inline int que(int x) {
		int res = 0;
		for (int i = x; i >= 1; i -= lowbit(i)) {
			res = std::max(res, t[i]);
		}
		return res;
	}
	inline void init() { memset(t, 0, sizeof(t)); }
}; // namespace fwt

int f[N + 5], g[N + 5], h[N + 5];

int pfx[N + 5], sfx[N + 5];

int main() {
	n = rd(), m = rd();
	for (int i = 1; i <= n; i++) {
		a[i] = rd();
		b[++bct] = a[i];
	}
	for (int i = 1; i <= m; i++) {
		int x = rd(), d = rd();
		q[x].emplace_back(d, i);
		b[++bct] = d;
	}
	std::sort(b + 1, b + 1 + bct);
	bct = std::unique(b + 1, b + 1 + bct) - b - 1;
	for (int i = 1; i <= n; i++) {
		a[i] = std::lower_bound(b + 1, b + 1 + bct, a[i]) - b;
		for (auto &[x, _] : q[i]) {
			x = std::lower_bound(b + 1, b + 1 + bct, x) - b;
		}
	}

	fwt::init();
	for (int i = 1; i <= n; i++) {
		f[i] = fwt::que(a[i] - 1) + 1;
		fwt::upd(a[i], f[i]);
	}
	fwt::init();
	for (int i = n; i >= 1; i--) {
		g[i] = fwt::que(bct - a[i]) + 1;
		fwt::upd(bct - a[i] + 1, g[i]);
	}

	int k = 0;
	for (int i = 1; i <= n; i++) k = std::max(k, f[i] + g[i] - 1);
	for (int i = 1; i <= n; i++) if (f[i] + g[i] - 1 == k) h[f[i]]++;
	fwt::init();
	for (int i = 1; i <= n; i++) {
		for (auto [x, id] : q[i]) {
			pfx[id] = fwt::que(x - 1);
		}
		fwt::upd(a[i], f[i]);
	}
	fwt::init();
	for (int i = n; i >= 1; i--) {
		for (auto [x, id] : q[i]) {
			sfx[id] = fwt::que(bct - x);
		}
		fwt::upd(bct - a[i] + 1, g[i]);
	}

	for (int i = 1; i <= n; i++) {
		for (auto [x, id] : q[i]) {
			if (pfx[id] + sfx[id] + 1 < k && f[i] + g[i] - 1 == k && h[f[i]] == 1) {
				ans[id] = k - 1;
			} else ans[id] = std::max(pfx[id] + sfx[id] + 1, k);
		}
	}
	for (int i = 1; i <= m; i++) printf("%d\n", ans[i]);
	return 0;
}
```

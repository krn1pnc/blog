---
title: Codeforces 1849F XOR Partition 题解
date: 2023-08-31 21:38:58
categories: 题解
tags:
  - 图论
  - 二分图
  - 位运算
---

## 思路

不妨二分答案．

考虑如何判定存在大于 $x$ 的划分方式，我们需要将异或和小于 $x$ 的数对划分到两个集合中．我们在这些点之间建边，判定是否是二分图即可．

由于边数是 $O(n^2)$ 的，直接做无法通过本题．

不妨将原序列排序，设排序之后的序列为 $a_i$，我们尝试证明一个引理：

> 若 $i$ 和 $i + 4$ 之间有连边，那么一定不合法．

若 $i$ 和 $i + 4$ 之间有连边，那么有 $a_i \oplus a_{i + 4} < x$．

考虑 $a_i \oplus a_{i + 4}$ 的最高位，我们将 $a_i \sim a_{i + 4}$ 的这一位单独拉出来，显然比这一位高的只会是 $0$，且这几位的情况只可能是 $00001$，$00011$，$00111$，$01111$ 这 $4$ 种，而前两种中，$i \sim i + 2$ 这三之间两两都可以连边，后两种中，$i + 2 \sim i + 4$ 这三之间两两也可以连边．

于是我们只需要考虑 $i$ 与 $[i + 1, i + 3]$ 这个区间中的点的连边．边数 $O(n)$，总复杂度 $O(n \log n)$．

## 代码

```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <cstring>
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

const int N = 2e5;

int n;
struct ele {
	int x, pos;
} a[N + 5];
std::vector<int> g[N + 5];

int col[N + 5], flg;
void dfs(int u) {
	for (int v : g[u]) {
		if (col[v] == -1) col[v] = col[u] ^ 1, dfs(v);
		else if (col[v] == col[u]) flg = false;
	}
}

bool check(int x) {
	for (int i = 1; i <= n; i++) g[i].clear();
	for (int i = 1; i <= n; i++) {
		for (int j = i + 1, li = std::min(n, i + 3); j <= li; j++) {
			if ((a[i].x ^ a[j].x) < x) g[i].push_back(j), g[j].push_back(i);
		}
	}
	memset(col, -1, sizeof(col));
	for (int i = 1; i <= n; i++) {
		if (col[i] == -1) {
			col[i] = 0, flg = true, dfs(i);
			if (!flg) return false;
		}
	}
	return true;
}

int ans[N + 5];

int main() {
	n = rd();
	for (int i = 1; i <= n; i++) a[i] = {rd(), i};
	std::sort(a + 1, a + 1 + n, [](ele a, ele b) { return a.x < b.x; });

	if (n == 2) return puts("10"), 0;

	int l = 0, r = (1 << 30) - 1, res = -1;
	while (l <= r) {
		int mid = (l + r) / 2;
		if (check(mid)) {
			l = mid + 1;
			res = mid;
		} else r = mid - 1;
	}
	check(res);
	for (int i = 1; i <= n; i++) ans[a[i].pos] = col[i];
	for (int i = 1; i <= n; i++) printf("%d", ans[i]);
	return 0;
}
```

## 参考

Purslane, [_CF1849F_](https://www.luogu.com.cn/blog/120947/solution-cf1849f)

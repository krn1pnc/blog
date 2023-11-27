---
title: Codeforces 1861F Four Suits 题解
date: 2023-09-05 13:48:19
categories: 题解
tags:
  - 网络流
  - 最小割
---

## 思路

考虑如何求每个人的答案．

设我们当前求答案的人为 $x$，不妨枚举取到最大值的花色 $y$，显然 $y$ 花色的牌应当尽量分给 $x$．现在我们要最小化其他人丢牌后剩下牌的最大值．

不妨二分答案 $\xi$．钦定第 $i$ 个人第 $j$ 中花色的牌最多只能拿 $\xi - a_{i, j}$ 张，发现这好像是个匹配问题！这不是我们网络最大流的题目吗！于是建立二分图模型，流量代表牌．左部有 $4$ 个点，代表 $4$ 种花色的牌，记 ${b^\prime}_i$ 表示给 $x$ 分配完后牌堆里剩下 $i$ 花色的牌数，那么左部 $i$ 号点的流量就是 ${b^\prime}_i$．右部有 $n - 1$ 个点，代表现在还没有被分配牌的 $n - 1$ 个人，记 $\mathrm{need}_i$ 代表 $i$ 还需要被发的牌数，右部 $i$ 号点的流量就是 $\mathrm{need}_i$．由于某种花色的牌最多只能拿 $\xi$ 张，所以左部 $j$ 号点到右部 $i$ 号点的流量就是 $\xi - a_{i, j}$．一个 $\xi$ 合法的判定条件是这个网络的最大流等于 $\sum_i \mathrm{need}_i$．

直接 Dinic 显然跑不了一点，于是考虑优化．

由最大流最小割定理，我们尝试计算这个图的最小割．由于左部点数很小，我们可以直接枚举左部有哪些点 $i$ 被右部点割掉，记这个集合为 $S$．考虑如何计算右部点割掉 $S$ 的最小代价，每个右部点可以割掉连向左部点的边或割掉和汇点的边，那么这个割的代价就是

$$
\sum_{i \not\in S} {b^\prime}_i + \sum_{i \not= x} \min\left\{\mathrm{need}_i, \sum_{j \in S} \xi - a_{i, j}\right\}
$$

这样单次判定是 $\Theta(n \log n)$ 的，对每个人求一次，复杂度就变成了 $\Theta(n^2 \log n)$，仍然不可接受．

发现复杂度瓶颈在计算 $\sum\min$ 这一项．这个 $\min$ 相当烦人，想办法去掉．

考察什么情况下才会取 $\mathrm{need}_i$，稍稍推导一下性质：

$$
\begin{aligned}
  \mathrm{need}_i &< \sum_{j \in S} \xi - a_{i, j} \\
  \mathrm{need}_i &< |S| \times \xi - \sum_{j \in S} a_{i, j} \\
  \mathrm{need}_i + \sum_{j \in S} a_{i, j} &< |S| \times \xi  \\
\end{aligned}
$$

发现如果把右部点按照 $\mathrm{need}_i + \sum_{j \in S} a_{i, j}$ 排序，那么取的就是一段前缀，且结束位置可以二分．

预处理一堆东西之后就做完了．复杂度 $\Theta(n \log^2 n)$．枚举的集合个数是 $2^4 = 16$，算常数．

## 代码

```cpp
#include <algorithm>
#include <bitset>
#include <cassert>
#include <cstdio>
#define debug(...) fprintf(stderr, __VA_ARGS__)

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

const int N = 5e4;

int n, m, a[N + 5][4], b[4];

int sumneed, need[N + 5];

int premx[N + 5], sufmx[N + 5];

int sum[1 << 4][N + 5];
int seq[1 << 4][N + 5], pos[1 << 4][N + 5], rpos[1 << 4][N + 5];
int pres[1 << 4][N + 5], sufs[1 << 4][N + 5];

int solve(int x, int y) {
	int used = std::min(need[x], b[y]);
	b[y] -= used;

	int mx = a[x][y] + used;
	int l = std::max(premx[x - 1], sufmx[x + 1]), r = 1e9, res = 1e9;
	auto check = [&](int alpha) {
		int mnc = 1e9;
		for (int S = 0; S < (1 << 4); S++) {
			int cur = 0;
			for (int i = 0; i < 4; i++) {
				if (!(S & (1 << i))) cur += b[i];
			}

			int siz = __builtin_popcount(S);
			int p = std::lower_bound(pos[S] + 1, pos[S] + 1 + n, siz * alpha, [&](int i, int x) { return seq[S][i] < x; }) - pos[S];
			cur += pres[S][p - 1] + (n - p + 1) * siz * alpha - sufs[S][p];
			if (rpos[S][x] < p) cur -= need[x];
			else cur -= siz * alpha - sum[S][x];

			mnc = std::min(mnc, cur);
		}
		return mnc == sumneed - need[x];
	};
	while (l <= r) {
		int mid = (l + r) / 2;
		if (check(mid)) r = mid - 1, res = mid;
		else l = mid + 1;
	}

	b[y] += used;
	return mx - res;
}

int main() {
	n = rd();
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j < 4; j++) m += (a[i][j] = rd());
	}
	for (int i = 0; i < 4; i++) m += (b[i] = rd());
	assert(m % n == 0), m /= n;

	for (int i = 1; i <= n; i++) {
		int ct = 0;
		for (int j = 0; j < 4; j++) ct += a[i][j];
		sumneed += (need[i] = m - ct);
	}
	for (int i = 1; i <= n; i++) {
		int cur = 0;
		for (int j = 0; j < 4; j++) cur = std::max(cur, a[i][j]);
		premx[i] = std::max(premx[i - 1], cur);
	}
	for (int i = n; i >= 1; i--) {
		int cur = 0;
		for (int j = 0; j < 4; j++) cur = std::max(cur, a[i][j]);
		sufmx[i] = std::max(sufmx[i + 1], cur);
	}
	for (int S = 0; S < (1 << 4); S++) {
		for (int i = 1; i <= n; i++) {
			for (int j = 0; j < 4; j++) {
				if (S & (1 << j)) sum[S][i] += a[i][j];
			}
		}
	}
	for (int S = 0; S < (1 << 4); S++) {
		for (int i = 1; i <= n; i++) seq[S][i] = sum[S][i] + need[i], pos[S][i] = i;
		std::sort(pos[S] + 1, pos[S] + 1 + n, [&](int i, int j) { return seq[S][i] < seq[S][j]; });
		for (int i = 1; i <= n; i++) rpos[S][pos[S][i]] = i;
		for (int i = 1; i <= n; i++) pres[S][i] = pres[S][i - 1] + need[pos[S][i]];
		for (int i = n; i >= 1; i--) sufs[S][i] = sufs[S][i + 1] + sum[S][pos[S][i]];
	}

	for (int i = 1; i <= n; i++) {
		int ans = 0;
		for (int j = 0; j < 4; j++) {
			ans = std::max(ans, solve(i, j));
		}
		printf("%d%c", ans, " \n"[i == n]);
	}
	return 0;
}
```

## 参考

jiangly, [_Educational Codeforces Round 154 (Rated for Div. 2)_](https://www.luogu.com.cn/blog/jiangly/educational-codeforces-round-154-rated-for-div-2-post)

---
title: 2023.3.16 省选模拟赛 景中人 (scene) 题解
date: 2023-03-21 22:33:23
categories: 题解
tags:
  - 动态规划
---

## 题面

### 题目描述

有 $n$ 个看风景的人在桥上．桥可以看成一个二维平面，那么每个人的位置都可以用一个坐标来表示．

Yazid 突发奇想，想用矩形把他们都覆盖住．

Yazid 发现，只需要用 $1$ 个巨大的矩形就可以做到这点．于是他规定单个矩形的面积不能超过 $S$．

Yazid 还觉得桥的下边的栏杆很优秀，于是他又规定了矩形的一条边必须贴着下栏杆（直线 $y = 0$）．

这下可把辣鸡蒟蒻 Yazid 难住了，他找到了即将参加 NOI 的你，请你告诉他，他至少要用几个矩形才能覆盖所有的景中人呢？

### 数据范围

$n \le 100$，$S \le 2 \times 10^5$．

## 思路

容易发现 $x$ 坐标相同的情况下，只有 $y$ 坐标最大的点才需要纳入考虑，所以可以把其他点直接丢掉．记丢完点后按 $x$ 坐标升序排序后第 $i$ 个点为 $(x_i, y_i)$．

我们考虑使用区间 DP 来解决这个问题．设 $f_{l,r}$ 表示覆盖 $[l,r]$ 之内的所有点的最小矩形数．对于一个区间 $[l,r]$，其矩形的最大高度是确定的，记为 $h = S / (x_l - x_r)$．

考虑转移：

- 两个区间是可以无代价合并的，所以枚举断点 $k \in [l,r)$，我们有 $f_{l,r} = \min\{f_{l,k} + f_{k + 1,r}\}$．
- 找到第一个和最后一个满足 $y > h$ 的点，记作 $p,q$，有 $f_{l,r} = f_{p,q} + 1$．若不存在这样的点，则显然有 $f_{l,r} = 1$．
- 对所有 $y > h$ 的点做一次 DP，计算出覆盖它们需要多少矩形．记这样的点有 $c$ 个，设 $g_i$ 表示考虑到了第 $i$ 个 $y > h$ 的点，转移显然．有 $f_{l,r} = g_c + 1$．

转移对以上三种情况取 $\min$ 即可，边界条件显然有 $f_{i,i} = 1$．

时间复杂度 $O(n^4)$．

## 代码

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <vector>

constexpr int N = 100;

int T;
int n, S;

struct point {
	int x, y;
} a[N + 5];

int f[N + 5][N + 5];

int main() {
	scanf("%d", &T);
	while (T--) {
		memset(f, 0, sizeof(f));

		scanf("%d%d", &n, &S);
		for (int i = 1; i <= n; i++) scanf("%d%d", &a[i].x, &a[i].y);

		std::sort(a + 1, a + 1 + n, [](point a, point b) { return a.x != b.x ? a.x < b.x : a.y > b.y; });
		n = std::unique(a + 1, a + 1 + n, [](point a, point b) { return a.x == b.x; }) - a - 1;

		for (int i = 1; i <= n; i++) f[i][i] = 1;
		for (int len = 2; len <= n; len++) {
			for (int l = 1, r = len; r <= n; l++, r++) {
				f[l][r] = 1e9;
				int mxh = S / (a[r].x - a[l].x);
				int p = l, q = r;
				while (p <= r && a[p].y <= mxh) p++;
				while (q >= l && a[q].y <= mxh) q--;
				if (p <= q) {
					f[l][r] = f[p][q] + 1;
					for (int i = l; i <= r - 1; i++) {
						f[l][r] = std::min(f[l][r], f[l][i] + f[i + 1][r]);
					}
					std::vector<int> ex{0}, g{0};
					for (int i = l; i <= r; i++) {
						if (a[i].y <= mxh) continue;
						ex.push_back(i), g.push_back(1e9);
					}
					for (int i = 1, li = ex.size() - 1; i <= li; i++) {
						for (int j = 1; j <= i; j++) {
							g[i] = std::min(g[i], g[j - 1] + f[ex[j]][ex[i]]);
						}
					}
					f[l][r] = std::min(f[l][r], g.back() + 1);
				} else f[l][r] = 1;
			}
		}

		printf("%d\n", f[1][n]);
	}
	return 0;
}
```

## 参考

Thunder_S, [_【NOI2017模拟7.1】景中人_](https://www.cnblogs.com/Livingston/p/15827976.html)

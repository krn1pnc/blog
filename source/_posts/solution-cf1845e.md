---
title: Codeforces 1845E Boxes and Balls 题解
date: 2023-07-01 13:37:25
categories: 题解
tags:
  - 动态规划
  - 计数
---

## 思路

考虑如何对一个局面计算所需的最小操作次数．

通过观察可以发现，在操作次数最小的方案中，若将球视为互异的，移动前后所有小球位置的偏序关系不会改变．这很好理解，若在某个方案中，某两个小球移动后位置的偏序关系改变了，那么我们可以通过交换这两个小球的目的地，使操作次数减小．

若设在操作次数最小的方案中，小球 $i$ 的初始位置是 $s_i$，目的地是 $t_i$，那么操作次数显然是 $\sum_i |s_i - t_i|$．很可惜这个限制似乎没法刻画序列的性质，于是我们转而考虑序列中的每一个位置对操作次数造成的贡献．

贡献有两种，从前往后移动经过 $i$ 的和从后往前移动经过 $i$ 的．显然两种贡献不能同时存在，那么先考虑第一种贡献，第二种同理可以计算．

第一种贡献统计的是满足 $s_j \le i < t_j$ 的 $j$ 的数量．那么将空盒子看作 $0$，装了球的盒子看作 $1$，设初始局面的对应的序列为 $a_i$，结束局面对应的序列为 $b_i$，那么 $i$ 位置的贡献就是 $\sum\limits_{j = 1}^i a_i - \sum\limits_{j = 1}^i b_i$．

令 $p_i = \sum\limits_{j = 1}^i a_i$，$q_i = \sum\limits_{j = 1}^i b_i$，获得这个局面所需最小的操作次数就是 $\sum_i |p_i - q_i|$．

现在要对局面计数，那么我们设 $f_{i, j, r}$ 表示考虑了前 $i$ 个位置，满足 $q_i = j, \sum_x |p_x - q_x| = r$ 的方案数．考虑当前位置放不放球，转移如下：

$$
f_{i, j, r} = f_{i - 1, j - 1, r - |p_i - j|} + f_{i - 1, j, r - |p_i - j|}
$$

由于我们可以使某个球反复横跳来浪费操作次数，那么答案为：

$$
\sum_{j \equiv k \pmod 2} f_{n, p_n, j}
$$

这样做的时间复杂度是 $O(nk^2)$ 的，难以（并不是无法！）通过本题．

然后 ~~鬼知道怎么~~ 发现，有用的转移的 $|p_i - j|$ 是 $O(\sqrt{k})$ 级别的．缩小枚举范围即可做到 $O(nk^{1.5})$．

## 代码

记得滚数组，不然空间寄了．

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

using i64 = long long;
const i64 P = 1e9 + 7;
const int N = 1.5e3;
const int LIM = 40;

int n, k, a[N + 5];
int s[N + 5];
i64 f[2][N + 5][N + 5];

int main() {
	n = rd(), k = rd();
	for (int i = 1; i <= n; i++) a[i] = rd();
	for (int i = 1; i <= n; i++) s[i] = s[i - 1] + a[i];
	f[0][0][0] = 1;
	for (int i = 1; i <= n; i++) {
		memset(f[i & 1], 0, sizeof(f[i & 1]));
		for (int j = std::max(0, s[i] - LIM); j <= std::min(s[n], s[i] + LIM); j++) {
			int cost = std::abs(s[i] - j);
			for (int p = cost; p <= k; p++) {
				if (j) (f[i & 1][j][p] += f[(i - 1) & 1][j - 1][p - cost]) %= P;
				(f[i & 1][j][p] += f[(i - 1) & 1][j][p - cost]) %= P;
			}
		}
	}
	i64 ans = 0;
	for (int i = k; i >= 0; i -= 2) (ans += f[n & 1][s[n]][i]) %= P;
	printf("%lld\n", ans);
	return 0;
}
```

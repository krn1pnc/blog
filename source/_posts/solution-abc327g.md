---
title: ABC 327G Many Good Tuple Problems 题解
date: 2023-11-05 18:51:30
categories: 题解
tags:
  - 图论
  - 计数
  - 二分图
---

## 思路

若建无向边 $(S_i, T_i)$，那么序列对 $(S, T)$ 为好的当且仅当得到的图为二分图．

直接计数需要处理重边和边的排列，不是很好考虑．考虑求出 $n$ 个点 $m$ 条边的简单二分图数目，记作 $f_{n, m}$，枚举本质不同的边数 $i$，答案由下式给出：

$$
2^m \sum_i c_{m, i} \times f_{n, i}
$$

其中 $c_{m, i}$ 表示将 $m$ 个互不相同的小球放入 $i$ 个互不相同的盒子中，要求盒子非空的方案数．

考虑如何求出 $c_{n, m}$．这个问题是经典的，可以利用二项式反演得出：

$$
m^n = \sum_i \binom{m}{i} c_{n, i} \iff c_{n, m} = \sum_i (-1)^{m - i} \binom{m}{i} i^n
$$

考虑如何求出 $f_{n, m}$．有一个很 naive 的想法是，由于二分图可以被黑白染色，直接枚举黑色节点数 $i$，此时二部可以连 $i(n - i)$ 条边，总方案数为：

$$
\sum_i \binom{n}{i} \binom{i(n - i)}{m}
$$

不妨记上式为 $g_{n, m}$，实际上我们计算的是将 $n$ 个点黑白染色后，连 $m$ 条边使得边的两端颜色不同的方案数．由于每个联通块都有两种染色方式，所以对于一个有 $k$ 个联通块的二分图，它被计算了 $2^k$ 次，太坏了！

若我们令这种图的联通块个数为常数 $k$，可以直接将方案数除去 $2^k$ 得到二分图个数，而最简单的方式是令 $k = 1$，即对联通的这种图计数．不妨设将 $n$ 个点黑白染色后，连 $m$ 条边使得边的两端颜色不同，且图联通的方案数为 $h_{n, m}$．考虑容斥掉不合法的方案数，有：

$$
h_{n, m} = g_{n, m} - \sum_i \sum_j \binom{n - 1}{i - 1} h_{i, j} g_{n - i, m - j}
$$

其中 $i$ 枚举的是 $1$ 所在的联通块大小，$j$ 枚举的是这个联通块的边数，二项式系数的含义是从去除 $1$ 的 $n - 1$ 个编号中选出 $i - 1$ 个编号，分配给 $1$ 所在的联通块中的其他点．

现在 $n$ 个点 $m$ 条边的联通二分图个数就是 $\frac{1}{2} h_{n, m}$，那么 $f_{n, m}$ 的计算也可以通过枚举 $1$ 所在的联通块大小得到：

$$
f_{n, m} = \frac{h_{n, m}}{2} + \sum_i \sum_j \binom{n - 1}{i - 1} \frac{h_{i, j}}{2} f_{n - i, m - j}
$$

## 代码

```cpp
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

using i64 = long long;
const i64 P = 998244353;
const i64 i2 = (P + 1) / 2;

i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for (; p; b = b * b % P, p >>= 1) {
		if (p & 1) res = res * b % P;
	}
	return res;
}

const int N = 30;
const int M = N * N;

i64 C[M + 5][M + 5], S[M + 5][M + 5];
i64 fac[M + 5];
void pre() {
	for (int i = 0; i <= M; i++) C[i][0] = 1;
	for (int i = 1; i <= M; i++) {
		for (int j = 1; j <= M; j++) {
			C[i][j] = (C[i - 1][j - 1] + C[i - 1][j]) % P;
		}
	}
}

int n, m;
i64 f[N + 5][M + 5], g[N + 5][M + 5], h[N + 5][M + 5];

int main() {
	pre();

	n = rd(), m = rd();
	int lim = (n / 2) * ((n + 1) / 2);
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j <= lim; j++) {
			for (int k = 0; k <= i; k++) {
				(g[i][j] += C[i][k] * C[k * (i - k)][j] % P) %= P;
			}
		}
	}
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j <= lim; j++) {
			h[i][j] = g[i][j];
			for (int k = 1; k < i; k++) {
				for (int p = 0; p <= j; p++) {
					(h[i][j] += P - C[i - 1][k - 1] * h[k][p] % P * g[i - k][j - p] % P) %= P;
				}
			}
		}
	}
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j <= lim; j++) {
			(h[i][j] *= i2) %= P;
		}
	}
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j <= lim; j++) {
			f[i][j] = h[i][j];
			for (int k = 1; k < n; k++) {
				for (int p = 0; p <= j; p++) {
					(f[i][j] += C[i - 1][k - 1] * h[k][p] % P * f[i - k][j - p] % P) %= P;
				}
			}
		}
	}
	i64 ans = 0;
	for (int i = 0; i <= lim; i++) {
		i64 coef = 0;
		for (int j = 0; j <= i; j++) {
			if ((i - j) & 1) (coef += P - C[i][j] * fpow(j, m) % P) %= P;
			else (coef += C[i][j] * fpow(j, m) % P) %= P;
		}
		(ans += coef * f[n][i] % P) %= P;
	}
	printf("%lld\n", ans * fpow(2, m) % P);
	return 0;
}
```

## 参考

Nyaan, [_G - Many Good Tuple Problems Editorial_](https://atcoder.jp/contests/abc327/editorial/7557)

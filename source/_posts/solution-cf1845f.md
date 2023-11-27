---
title: Codeforces 1845F Swimmers in the Pool 题解
date: 2023-07-01 22:27:16
categories: 题解
tags:
  - 快速 Fourier 变换
  - 动态规划
  - 计数
---

## 思路

由物理知识可得，两个人 $i$ 和 $j$ 的相逢时刻只可能形如 $\frac{k \times 2l}{|v_i \pm v_j|}, k \in \N^*$．题意就是计算这样的有理数数量．

考虑计算出 $|v_i \pm v_j|$ 的所有可能的值．这是经典问题，若我们设 $F(x) = \sum_i x^{a_i}$，$G(x) = \sum_i x^{-a_i}$，那么 $[x^n](F * G)(x)$ 就是相减等于 $n$ 的二元组数量，相加的同理．

记 $U = \{|v_i \pm v_j|\}$，又由于分子上都有 $2l$，那么问题可以转化为计算形如 $x = \frac{k}{w}, w \in U$ 且满足 $x \le \frac{t}{2l}$ 的有理数 $x$ 的数目．对于某个 $w$，这样的分数数目显然是 $\left\lfloor \frac{tw}{2l} \right\rfloor$，但直接求和显然会有重复贡献．

考虑如何去掉重复贡献．对于每个 $w$，如果我们只计算以其为分母的最简分数数量，记这个数量为 $f_w$，那么计算不同有理数个数时，$w$ 的每个因数 $d$ 都会对答案恰好贡献 $f_d$．

考虑 $f_i$ 的计算．我们需要从不考虑限制的 $\left\lfloor \frac{ti}{2l} \right\rfloor$ 个分数中去除非最简分数，而这些非最简分数的最简分数形式的分母肯定是 $i$ 的一个因数．枚举 $i$ 的因数并去除这些非最简分数，我们可以得到以下转移：

$$
f_i = \left\lfloor \frac{ti}{2l} \right\rfloor - \sum_{d | i} f_d
$$

若记 $V$ 表示 $U$ 中元素所有因数构成的集合，答案即为 $\sum_{d \in V} f_d$．

转移复杂度是调和级数求和级别，计算 $U$ 使用 NTT 优化卷积即可．时间复杂度 $O(n \log n)$．

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

using i64 = long long;
const int B = 2e5;
const int N = 1 << 20;
const i64 Mod = 1e9 + 7;

const i64 P = 998244353;
const i64 G = 3;
const i64 iG = 332748118;
i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for (; p; b = b * b % P, p >>= 1) {
		if (p & 1) res = res * b % P;
	}
	return res;
}
namespace ntt {
	int lim, r[N];
	int init(int n) {
		lim = 1;
		while (lim < n) lim <<= 1;
		for (int i = 0; i < lim; i++) {
			r[i] = (r[i >> 1] >> 1) | ((i & 1) ? lim >> 1 : 0);
		}
		return lim;
	}
	void run(i64 *f, bool inv) {
		for (int i = 0; i < lim; i++) {
			if (i < r[i]) std::swap(f[i], f[r[i]]);
		}
		for (int mid = 1; mid < lim; mid *= 2) {
			i64 wn = fpow(inv ? iG : G, (P - 1) / (mid * 2));
			for (int i = 0; i < lim; i += (mid * 2)) {
				i64 w = 1;
				for (int j = 0; j < mid; j++, w = w * wn % P) {
					i64 a = f[i + j], b = w * f[i + mid + j] % P;
					f[i + j] = (a + b) % P, f[i + mid + j] = (a - b + P) % P;
				}
			}
		}
		if (inv) {
			i64 il = fpow(lim, P - 2);
			for (int i = 0; i < lim; i++) f[i] = f[i] * il % P;
		}
	}
};

i64 l, t, n, a[N];
i64 f[N], g[N];
bool flg[N];
i64 dp[N];

int main() {
	l = rd(), t = rd(), n = rd();
	for (int i = 1; i <= n; i++) a[i] = rd();

	for (int i = 1; i <= B * 2; i++) {
		(dp[i] += ((t * i) / (2 * l)) % Mod) %= Mod;
		for (int j = i + i; j <= B * 2; j += i) {
			(dp[j] += Mod - dp[i]) %= Mod;
		}
	}

	for (int i = 1; i <= n; i++) f[a[i]] = 1;
	for (int i = 1; i <= n; i++) g[-a[i] + B] = 1;
	int lim = ntt::init(B * 2);
	ntt::run(f, 0), ntt::run(g, 0);
	for (int i = 0; i < lim; i++) g[i] = g[i] * f[i] % P;
	for (int i = 0; i < lim; i++) f[i] = f[i] * f[i] % P;
	ntt::run(f, 1), ntt::run(g, 1);
	for (int i = 1; i <= n; i++) f[a[i] + a[i]]--;
	g[B] -= n;
	for (int i = 1; i <= B * 2; i++) {
		if (f[i]) flg[i] = 1;
		if (g[i]) flg[std::abs(i - B)] = 1;
	}
	for (int i = 1; i <= B * 2; i++) {
		for (int j = i + i; j <= B * 2; j += i) {
			flg[i] |= flg[j];
		}
	}

	i64 ans = 0;
	for (int i = 1; i <= B * 2; i++) {
		if (flg[i]) (ans += dp[i]) %= Mod;
	}
	printf("%lld\n", ans);
	return 0;
}
```

## 参考

PEKKA_l, [_CF1845F 题解_](https://www.luogu.com.cn/blog/PEKKA-l-2021/solution-cf1845f)

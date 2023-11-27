---
title: LOJ 6503 「雅礼集训 2018 Day4」Magic 题解
date: 2023-07-07 22:44:35
categories: 题解
tags:
  - 数学
  - 二项式反演
  - 动态规划
  - 快速 Fourier 变换
---

## 思路

先标个号，最后再给答案除 $\prod_i a_i!$．

直接计算恰好 $k$ 个魔术对的序列数量貌似困难，考虑二项式反演．若设 $f(x)$ 表示钦定含有 $x$ 对魔术对的序列数量，$g(x)$ 表示恰好含有 $x$ 对魔术对的序列数量，我们有：

$$
f(k) = \sum_{i = k}^n \binom{i}{k} g(i) \iff g(k) = \sum_{i = k}^n (-1)^{i - k} \binom{i}{k} f(i)
$$

考虑计算 $f(i)$．设 $f_{i, j}$ 表示考虑了前 $i$ 种牌，已经钦定了 $j$ 种魔术对的方案数．由于我们只确定了魔术对部分的方案数，剩下的牌可以瞎排，有 $f(i) = f_{m, i} \times (n - i)!$．

转移时枚举当前颜色的魔术对个数 $p$．我们可以从 $a_i$ 张这种牌中选出 $p$ 张来充当魔术对的第二张牌，这部分的方案数为 $\binom{a_i}{p}$．考虑将这 $p$ 张牌插到同种牌后面，这部分的方案数为 $(a_i - p)^{\overline{p}}$．转移方程如下：

$$
f_{i, j} = \sum_{0 \le p < a_i} \binom{a_i}{p} (a_i - p)^{\overline{p}} f_{i - 1, j - p}
$$

这是一个卷积的形式，考虑设

$$
F_i(x) = \sum_{0 \le p < a_i} \binom{a_i}{p} (a_i - p)^{\overline{p}} x^p
$$

，那么有

$$
f_{m, n} = [x^n]\prod_{i = 1}^m F_i(x)
$$

分治 FFT 即可．时间复杂度 $O(n \log^2 n)$．

## 代码

```cpp
#include <cstdio>
#include <vector>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

const int N = 1 << 20;

using i64 = long long;
const i64 P = 998244353;
const i64 G = 3;
const i64 iG = 332748118;

i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for(; p; b = b * b % P, p >>= 1) {
		if(p & 1) res = res * b % P;
	}
	return res;
}

using Poly = std::vector<i64>;

namespace ntt {
	int lim, r[N];
	int init(int n) {
		lim = 1; while(lim < n) lim <<= 1;
		for(int i = 0; i < lim; i++) {
			r[i] = (r[i >> 1] >> 1) | ((i & 1) ? lim >> 1 : 0);
		}
		return lim;
	}
	void run(Poly &f, bool inv) {
		for(int i = 0; i < lim; i++) {
			if(i < r[i]) std::swap(f[i], f[r[i]]);
		}
		for(int len = 1; len < lim; len *= 2) {
			i64 wn = fpow(inv ? iG : G, (P - 1) / (len * 2));
			for(int i = 0; i < lim; i += len * 2) {
				i64 w = 1;
				for(int j = 0; j < len; j++, (w *= wn) %= P) {
					i64 x = f[i + j], y = w * f[i + len + j] % P;
					f[i + j] = (x + y) % P, f[i + len + j] = (x - y + P) % P;
				}
			}
		}
		if(inv) {
			i64 il = fpow(lim, P - 2);
			for(int i = 0; i < lim; i++) (f[i] *= il) %= P;
		}
	}
};

i64 fac[N], ifac[N];
void pre(int n) {
	fac[0] = 1; for(int i = 1; i <= n; i++) fac[i] = fac[i - 1] * i % P;
	ifac[n] = fpow(fac[n], P - 2); for(int i = n - 1; i >= 0; i--) ifac[i] = ifac[i + 1] * (i + 1) % P;
}
i64 C(int n, int m) {
	if(n < 0 || m < 0 || n < m) return 0;
	return fac[n] * ifac[m] % P * ifac[n - m] % P;
}

int m, n, k, a[N];
Poly F[N], f;

void solve(int l, int r, Poly &f, int d) {
	if(l == r) return f = F[l], void();

	int mid = -1, c = 0;
	for(int i = l; i <= r; i++) {
		c += a[i];
		if(c * 2 >= d || i == r - 1) {
			mid = i; break;
		}
	}

	Poly L, R;
	solve(l, mid, L, c), solve(mid + 1, r, R, d - c);

	int lim = ntt::init(d); f.resize(lim);
	L.resize(lim), R.resize(lim);
	ntt::run(L, 0), ntt::run(R, 0);
	for(int i = 0; i < lim; i++) f[i] = L[i] * R[i] % P;
	ntt::run(f, 1);
}

i64 g[N];

int main() {
	pre(N - 1);
	m = rd(), n = rd(), k = rd();
	for(int i = 1; i <= m; i++) a[i] = rd();
	for(int i = 1; i <= m; i++) {
		for(int b = 0; b < a[i]; b++) {
			F[i].push_back(C(a[i], b) * fac[a[i] - 1] % P * ifac[a[i] - b - 1] % P);
		}
	}

	solve(1, m, f, n);

	for(int i = 0; i <= n; i++) g[i] = f[i] * fac[n - i] % P;
	i64 ans = 0;
	for(int i = k; i <= n; i++) {
		if((i - k) & 1) (ans += P - C(i, k) * g[i] % P) %= P;
		else (ans += C(i, k) * g[i] % P) %= P;
	}
	for(int i = 1; i <= m; i++) (ans *= ifac[a[i]]) %= P;
	printf("%lld\n", ans);
	return 0;
}
```

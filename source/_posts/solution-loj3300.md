---
title: LOJ 3300 「联合省选 2020 A」组合数问题 题解
date: 2023-05-12 22:44:35
categories: 题解
tags:
  - 数学
  - 第二类 Stirling 数
---

## 思路

众所周知，组合数和下降幂放在一起会发生奇妙的反应．于是尝试将 $f(k)$ 拆开得到：

$$
f(k) = \sum_{i = 0}^m a_ik^i = \sum_{i = 0}^m a_i \sum_{j = 0}^i {i \brace j} k^{\underline{j}}
$$

带入原式可得：

$$
\begin{aligned}
  \sum_{k = 0}^n x^k \binom{n}{k} f(k)
  &= \sum_{k = 0}^n x^k \binom{n}{k} \sum_{i = 0}^m a_i \sum_{j = 0}^i {i \brace j} k^{\underline{j}} \\
  &= \sum_{k = 0}^n x^k \binom{n}{k} \sum_{j = 0}^m k^{\underline{j}} \sum_{i = j}^m {i \brace j} a_i
\end{aligned}
$$

记 $u_x = \sum\limits_{i = x}^m {i \brace x} a_i$，稍加变换可得：

$$
\sum_{k = 0}^n x^k \binom{n}{k} \sum_{j = 0}^m k^{\underline{j}} \sum_{i = j}^m {i \brace j} a_i = \sum_{j = 0}^m \sum_{k = 0}^n \binom{n}{k} k^{\underline{j}} x^k u_j
$$

又注意到：

$$
\begin{aligned}
  \binom{n}{k} k^{\underline{j}}
  &= \frac{n!}{k!(n - k)!} \frac{k!}{(k - j)!} \\
  &= \frac{n!}{(n - j)!} \frac{(n - j)!}{(k - j)!(n - k)!} \\
  &= n^{\underline{j}} \binom{n - j}{k - j}
\end{aligned}
$$

可得：

$$
\begin{aligned}
  \sum_{j = 0}^m \sum_{k = 0}^n \binom{n}{k} k^{\underline{j}} x^k u_j
  &= \sum_{j = 0}^m \sum_{k = 0}^n n^{\underline{j}} \binom{n - j}{k - j} x^k u_j \\
  &= \sum_{j = 0}^m n^{\underline{j}} u_j \sum_{k = 0}^n \binom{n - j}{k - j} x^k
\end{aligned}
$$

将指标 $k$ 换成 $k - j$，再应用二项式定理，我们得到：

$$
\begin{aligned}
  \sum_{j = 0}^m n^{\underline{j}} u_j \sum_{k = 0}^n \binom{n - j}{k - j} x^k
  &= \sum_{j = 0}^m n^{\underline{j}} u_j \sum_{k = 0}^{n - j} \binom{n - j}{k} x^{k + j} \\
  &= \sum_{j = 0}^m n^{\underline{j}} u_j x^j \sum_{k = 0}^{n - j} \binom{n - j}{k} x^k \\
  &= \sum_{j = 0}^m n^{\underline{j}} u_j x^j (x + 1)^{n - j}
\end{aligned}
$$

$O(m^2)$ 预处理第二类 Stirling 数后即可 $O(m^2)$ 或 $O(m \log n)$ 计算．

## 代码

我是懒狗，只写了 $O(m^2)$ 的．

```cpp
#include <cstdio>

using i64 = long long;

inline i64 rd() {
	i64 x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

const int N = 1e3;

i64 n, x, P, m, a[N + 5];
i64 S[N + 5][N + 5];
void pre() {
	S[0][0] = 1;
	for(int i = 1; i <= m; i++) {
		for(int j = 1; j <= m; j++) {
			S[i][j] = (S[i - 1][j - 1] + j * S[i - 1][j] % P) % P;
		}
	}
}

i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for(; p; b = b * b % P, p >>= 1) {
		if(p & 1) res = res * b % P;
	}
	return res;
}

int main() {
	n = rd(), x = rd(), P = rd(), m = rd(); pre();
	for(int i = 0; i <= m; i++) a[i] = rd();
	i64 ans = 0;
	for(int j = 0; j <= m; j++) {
		i64 nj = 1; for(int i = n - j + 1; i <= n; i++) (nj *= i) %= P;
		i64 fi = 0; for(int i = j; i <= m; i++) (fi += S[i][j] * a[i] % P) %= P;
		(ans += nj * fi % P * fpow(x, j) % P * fpow(x + 1, n - j) % P) %= P;
	}
	printf("%lld\n", ans);
	return 0;
}
```

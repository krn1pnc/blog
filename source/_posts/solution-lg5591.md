---
title: 洛谷 5591 小猪佩奇学数学 题解
date: 2023-07-08 22:21:03
categories: 题解
tags:
  - 数学
  - 单位根反演
---

## 思路

注意到：

$$
\left\lfloor \frac{i}{k} \right\rfloor = \frac{i - i \bmod k}{k}
$$

那么我们有：

$$
\begin{aligned}
  \text{原式}
  &= \sum_{i = 0}^n \binom{n}{i} \times p^i \times \frac{i - i \bmod k}{k} \\
  &= \frac{1}{k} \sum_{i = 0}^n \binom{n}{i} p^i (i - i \bmod k) \\
  &= \frac{1}{k} \left(\sum_{i = 0}^n \binom{n}{i} p^i i - \sum_{i = 0}^n \binom{n}{i} p^i (i \bmod k)\right) \\
\end{aligned}
$$

令 $a = \sum_{i = 0}^n \binom{n}{i} p^i i$，$b = \sum_{i = 0}^n \binom{n}{i} p^i (i \bmod k)$，分别考虑计算两部分．

1. 对于 $a$，注意到 $\binom{n}{i} = \binom{n - 1}{i - 1} \frac{n}{i}$，我们有：

   $$
   \begin{aligned}
     a
     &= n \sum_{i = 1}^n \binom{n - 1}{i - 1} p^i \\
     &= np \sum_{i = 0}^{n - 1} \binom{n - 1}{i} p^i \\
     &= np(p + 1)^{n - 1}
   \end{aligned}
   $$

   容易计算．

2. 对于 $b$，考虑枚举 $i \bmod k$ 的值，然后单位根反演：

   $$
   \begin{aligned}
     b
     &= \sum_{i = 0}^n \binom{n}{i} p^i \sum_{j = 0}^{k - 1} j[j \equiv i \pmod k] \\
     &= \sum_{i = 0}^n \binom{n}{i} p^i \sum_{j = 0}^{k - 1} j[k | i - j] \\
     &= \sum_{i = 0}^n \binom{n}{i} p^i \sum_{j = 0}^{k - 1} j \frac{1}{k} \sum_{r = 0}^{k - 1} {w_k}^{(i - j)r} \\
     &= \frac{1}{k} \sum_{r = 0}^{k - 1} \sum_{j = 0}^{k - 1} j{w_k}^{-jr} \sum_{i = 0}^n \binom{n}{i} p^i {w_k}^{ir} \\
     &= \frac{1}{k} \sum_{r = 0}^{k - 1} (p{w_k}^r + 1)^n \sum_{j = 0}^{k - 1} j{w_k}^{-jr} \\
   \end{aligned}
   $$

   发现后面那个 $\sum$ 是非常经典的求和！不妨设 $f(x) = \sum_{i = 0}^{k - 1} ix^i$，后面那个就是 $f({w_k}^{-j})$．错位相减可得：

   $$
   f(x) =
   \begin{cases}
     \frac{k(k - 1)}{2} & x = 1 \\
     \frac{k}{x - 1} & \text{otherwise}
   \end{cases}
   $$

   然后 $b$ 就能够快速计算了．

总时间复杂度 $O(k \log n)$．

## 代码

```cpp
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

const i64 P = 998244353;
const i64 G = 3;

i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for(; p; b = b * b % P, p >>= 1) {
		if(p & 1) res = res * b % P;
	}
	return res;
}
i64 inv(i64 x) { return fpow(x, P - 2); }

i64 n, p, k;

i64 f(i64 x) {
	if(x == 1) return k * (k - 1) % P * inv(2) % P;
	return k * inv((x - 1 + P) % P) % P;
}

int main() {
	n = rd(), p = rd(), k = rd();
	i64 wk = fpow(G, (P - 1) / k), w = 1, a = 0;
	for(int i = 0; i < k; i++, (w *= wk) %= P) {
		(a += fpow((p * w % P + 1) % P, n) * f(inv(w)) % P) %= P;
	}
	i64 b = n * p % P * fpow(p + 1, n - 1) % P;
	printf("%lld\n", (b - a * inv(k) % P + P) % P * inv(k) % P);
	return 0;
}
```

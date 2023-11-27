---
title: 洛谷 7386 「EZEC-6」0-1 Trie 题解
date: 2023-09-03 10:51:13
categories: 题解
tags:
  - 数学
  - 计数
  - Lucas 定理
---

## 思路

首先，利用插板法容易得出，题目中限制的合法串的数目是 $\binom{n - 1}{m - 1}$，且存在合法串的充要条件为 $n \le m$．

设 $\mathrm{dup}$ 表示所有串按照字典序排序后相邻两串的 LCP 长度求和．由于我们排序了，贡献不重不漏，故答案为 $(n + m)\binom{n - 1}{m - 1} - \mathrm{dup}$．

考虑如何求出 $\mathrm{dup}$，我们枚举 LCP 长度 $i$ 和 LCP 内 $1$ 的个数 $j$，排序后相邻两串一定形如 $\mathrm{lcp} + \texttt{0} + a$ 和 $\mathrm{lcp} + \texttt{1} + b$，不妨设为 $s$ 和 $t$．回想存在合法串的条件，我们要保证 $\mathrm{lcp}$ 和得到的两串均合法，带入相关变量，可以写下如下限制：

1. $\mathrm{lcp}$ 合法：$j + 1 \le i - j$．
2. $s$ 合法：$n - j \le (n + m - i) - (n - j)$．
3. $t$ 合法：$n - j - 1 \le (n + m - i - 1) - (n - j - 1)$．

整理可得：$2j + 1 \le i \le m - n + 2j$．又由于强制以 $\texttt{1}$ 结尾，故 $j$ 只能枚举到 $n - 2$．

那么开始大力推式：

$$
\begin{aligned}
  \mathrm{dup}
  &= \sum_{j = 0}^{n - 2} \sum_{i = 2j + 1}^{m - n + 2} \binom{i - j - 1}{j} \times i \\
  &= \sum_{j = 0}^{n - 2} \sum_{i = 2j + 1}^{m - n + 2} \binom{i - j - 1}{j} \times (i - j) + \binom{i - j - 1}{j} \times j \\
  &= \sum_{j = 0}^{n - 2} \sum_{i = 2j + 1}^{m - n + 2} \binom{i - j}{j + 1} \times (j + 1) + \binom{i - j - 1}{j} \times j \\
  &= \sum_{j = 0}^{n - 2} \binom{m - n + j + 1}{j + 2} \times (j + 1) + \binom{m - n + j}{j + 1} \times j \\
  &= (n - 1) \times \binom{m - 1}{n} + 2 \times \sum_{j = 0}^{n - 2} \binom{m - n + j}{j + 1} \times j \\
  &= (n - 1) \times \binom{m - 1}{n} + 2 \times \left((m - n) \times \binom{m}{m - n + 1} - (m - n + 1) \times \binom{m - 1}{m - n} + 1\right) \\
\end{aligned}
$$

这里就可以直接算了，当然也可以再化简一下．化简之后答案为 $2 \binom{m + 1}{n} - \binom{m - 1}{n} - 2$．

## 代码

```cpp
#include <cstdio>
#define debug(...) fprintf(stderr, __VA_ARGS__)

using i64 = long long;

inline i64 rd() {
	i64 x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

const i64 P = 18888913;

i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for (; p; b = b * b % P, p >>= 1) {
		if (p & 1) res = res * b % P;
	}
	return res;
}

i64 fac[P], ifac[P];
void pre() {
	fac[0] = 1;
	for (int i = 1; i < P; i++) fac[i] = fac[i - 1] * i % P;
	ifac[P - 1] = fpow(fac[P - 1], P - 2);
	for (int i = P - 2; i >= 0; i--) ifac[i] = ifac[i + 1] * (i + 1) % P;
}
i64 _C(int n, int m) {
	if (n < 0 || m < 0 || n < m) return 0;
	return fac[n] * ifac[m] % P * ifac[n - m] % P;
}
i64 C(i64 n, i64 m) {
	if (!m) return 1;
	return _C(n % P, m % P) * C(n / P, m / P) % P;
}

i64 solve() {
	i64 n = rd(), m = rd();
	if (n > m) return 0;
	return ((2 * C(m + 1, n) - C(m - 1, n) - 2) % P + P) % P;
}

int main() {
	pre();
	int T = rd(), ans = 0;
	while (T--) ans ^= solve();
	printf("%d\n", ans);
	return 0;
}
```

## 参考

Mivik, [_[题解] [EZEC-6] 0-1 Trie_](https://mivik.moe/2021/solution/ezec-01-trie/)

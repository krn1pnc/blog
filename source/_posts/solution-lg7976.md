---
title: 洛谷 7976 「Stoi2033」园游会 题解
date: 2023-09-23 15:17:37
categories: 题解
tags:
  - 数学
  - 计数
  - Lucas 定理
---

## 思路

先证明一下打表找出的规律．

> 引理：记 $i$ 的三进制表达中 $1$ 的个数为 $\mathrm{ct}$，有
>
> $$
> \sum_{j} \left(\binom{i}{j} + 1\right) \bmod 3 - 1 = 2^\mathrm{ct}
> $$

在我校 MO 爷的帮助下给出的证明：

设满足 $\binom{n}{m} \equiv 1 \pmod 3$ 的 $m$ 有 $a$ 个，满足 $\binom{n}{m} \equiv 2 \pmod 3$ 的 $m$ 有 $b$ 个，即证明 $a - b = 2^\mathrm{ct}$．

考虑对 $\binom{n}{m} \bmod 3$ 应用 Lucas 定理，设 $n, m$ 的三进制表达分别为 $(n_1 n_2 \cdots n_p)_3, (m_1 m_2 \cdots m_p)_3$，有 $\binom{n}{m} \equiv \prod_i \binom{n_i}{m_i} \pmod 3$．记 $r, s, t$ 分别为 $n_i$ 等于 $0, 1, 2$ 时 $i$ 构成的集合．

考虑计算 $a$．要使 $\binom{n}{m} \equiv 1 \pmod 3$，$m$ 需要满足下列条件：

- 由于 $r, s$ 中的位对乘积的贡献只能为 $1$，那么有 $\forall i \in r, m_i = 0$，$\forall i \in s, m_i \in \{0, 1\}$．
- 由于 $2^k \equiv 1 \pmod 3 \iff 2 \mid k$，有 $\sum\limits_{i \in t} [m_i = 1]$ 是偶数．

枚举有多少 $i \in t$ 使得 $m_i = 1$，有

$$
a = 2^{|s|} \sum_{k = 0}^{|t|} [2 \mid k] \binom{|t|}{k} 2^{|t| - k}
$$

同理可得

$$
b = 2^{|s|} \sum_{k = 0}^{|t|} [2 \nmid k] \binom{|t|}{k} 2^{|t| - k}
$$

那么有

$$
a - b = 2^{|s|} \sum_{k = 0}^{|t|} \binom{|t|}{k} (-1)^k 2^{|t| - k}
$$

这不是我们二项式定理的题目吗！于是有

$$
a - b = 2^{|s|} (2 - 1)^{|t|} = 2^{|s|} = 2^\mathrm{ct}
$$

得证．

然后就可以枚举 $\mathrm{ct}$，数位 DP 计算 $\le n$ 的数中有几个满足三进制表达中 $1$ 的个数为 $\mathrm{ct}$．

## 代码

卡了下常，写得很丑．

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#define debug(...) fprintf(stderr, __VA_ARGS__)

using u64 = unsigned long long;

inline u64 rd() {
	u64 x = 0; int c = getchar();
	while (((c - '0') | ('9' - c)) < 0) c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return x;
}

const u64 P = 1732073999;

const int N = 40;

int nn[N + 5], len;
u64 pw[N + 5];

u64 f[N + 5][N + 5][2];
u64 dfs(int i, int ct, bool rmx) {
	if (i == len + 1) return ct == 0;
	if (f[i][ct][rmx] != 0xffffffffffffffff) return f[i][ct][rmx];
	u64 res = 0;
	for (int cur = 0, li = (rmx ? nn[i] : 2); cur <= li; cur++) {
		int nc = ct - (cur == 1);
		if (nc < 0) continue;
		int nxt = dfs(i + 1, nc, rmx && cur == nn[i]);
		res += nxt;
	}
	return f[i][ct][rmx] = res % P;
}

void solve() {
	memset(f, 0xff, sizeof(f));

	u64 n = rd(); len = 0;
	while (n) nn[++len] = n % 3, n /= 3;
	std::reverse(nn + 1, nn + 1 + len);
	u64 ans = 0;
	for (int i = 0; i <= len; i++) {
		ans += dfs(1, i, 1) * pw[i] % P;
	}
	printf("%llu\n", ans % P);
}

int main() {
	pw[0] = 1; for (int i = 1; i <= N; i++) pw[i] = pw[i - 1] * 2 % P;
	int T = rd(); rd();
	while (T--) solve();
	return 0;
}
```

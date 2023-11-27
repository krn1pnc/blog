---
title: Codeforces 1768F Wonderful Jump 题解
date: 2023-07-19 23:07:35
categories: 题解
tags:
  - 根号分治
  - 动态规划
---

## 思路

首先有一个很简单的 1D / 1D DP，设 $f_i$ 表示从位置 $1$ 跳到位置 $i$ 的最小花费，有转移方程

$$
f_i = \min_{j < i} \{f_j + (i - j)^2 \times \min_{j \le k \le i} a_k\}
$$

状态已经最优了，转移的形式不像任何一种常见 DP 优化，那么考虑去掉无效转移．

注意到 $a_i \le n$，那么对于造成最终答案的方案，每一次从 $l$ 至 $l$ 的跳跃一定有

$$
(r - l)^2 \times \min_{l \le i \le r} a_i \le \sum_{i = l}^r a_i \le n \times (r - l)
$$

两边同时除去 $r - l$，有

$$
(r - l) \times \min_{l \le i \le r} a_i \le n
$$

看到这种相乘形式的约束，考虑根号分治．

1. $r - l \le \sqrt{n}$ 时，暴力枚举 $\sqrt{n}$ 个位置转移即可．
2. $\min_{l \le i \le r} a_i \le \sqrt{n}$ 时，考虑最小值在何处取到．

    首先需要再观察出一个性质，对于每一段，$\min$ 一定在端点处取得，否则可以通过将这一段在最值处砍成两端来做到更优．

    - $a_r$ 为最小值时，直接暴力枚举 $> a_l$ 的位置转移．
    - $a_l$ 为最小值时，可以维护 $\le \sqrt{n}$ 的每个值的最后出现位置，枚举这些值来转移．

总复杂度 $O(n \sqrt{n})$．

## 代码

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
 
inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}
 
using i64 = long long;
const int N = 4e5;
const int B = 700;
 
int n, a[N + 5];
i64 f[N + 5]; int lp[N + 5];
 
int main() {
	n = rd();
	for(int i = 1; i <= n; i++) a[i] = rd();
	memset(f, 63, sizeof(f)), f[1] = 0, lp[a[1]] = 1;
	for(int i = 2; i <= n; i++) {
		int mn = a[i];
		for(int j = i - 1; j >= std::max(1, i - B); j--) {
			mn = std::min(mn, a[j]);
			f[i] = std::min(f[i], f[j] + 1ll * mn * (i - j) * (i - j));
		}
		if(a[i] <= B) {
			for(int j = i - 1; j >= 1; j--) {
				if(a[j] <= a[i]) break;
				f[i] = std::min(f[i], f[j] + 1ll * a[i] * (i - j) * (i - j));
			}
		}
		for(int j = 1; j <= B; j++) {
			if(!lp[j]) continue;
			f[i] = std::min(f[i], f[lp[j]] + 1ll * j * (i - lp[j]) * (i - lp[j]));
		}
		lp[a[i]] = i;
	}
	for(int i = 1; i <= n; i++) printf("%lld%c", f[i], " \n"[i == n]);
	return 0;
}
```

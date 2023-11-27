---
title: LOJ 6044 「雅礼集训 2017 Day8」共 题解
date: 2023-06-16 22:36:38
categories: 题解
tags:
  - 数学
  - 计数
  - 矩阵树定理
---

## 思路

考虑钦定 $k$ 个点深度为奇数，这部分方案数为 $\binom{n - 1}{k - 1}$．

树是一个二分图，考虑将其黑白染色．我们可以发现深度为奇数的点颜色相同，于是剩下的部分转化成了计算完全二分图 $K_{k, n - k}$ 的生成树数目．我们有如下引理：

> $K_{n, m}$ 的生成树数目为 $n^{m - 1}m^{n - 1}$．

网上的大部分题解貌似都用的 Prüfer 序列进行证明，这里给出一种利用矩阵树定理的证明方式．

对 $K_{n, m}$ 应用矩阵树定理，其生成树个数为：

$$
\begin{vmatrix}
  m & 0 & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  0 & m & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & m & -1 & -1 & \cdots & -1 \\
  -1 & -1 & \cdots & - 1 & n & 0 & \cdots & 0 \\
  -1 & -1 & \cdots & - 1 & 0 & n & \cdots & 0 \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  -1 & -1 & \cdots & - 1 & 0 & 0 & \cdots & n \\
\end{vmatrix}
$$

其中，主对角线上为 $m$ 的元素一共有 $n - 1$ 个（去掉一行一列），为 $n$ 的元素有 $m$ 个．

使用前 $n - 1$ 行逐一消去左下区域内的 $-1$，原式可化为：

$$
\begin{vmatrix}
  m & 0 & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  0 & m & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & m & -1 & -1 & \cdots & -1 \\
  0 & 0 & \cdots & 0 & n - \frac{n - 1}{m} & -\frac{n - 1}{m} & \cdots & -\frac{n - 1}{m} \\
  0 & 0 & \cdots & 0 & -\frac{n - 1}{m} & n - \frac{n - 1}{m} & \cdots & -\frac{n - 1}{m} \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & 0 & -\frac{n - 1}{m} & -\frac{n - 1}{m} & \cdots & n - \frac{n - 1}{m} \\
\end{vmatrix}
$$

将最后 $m - 1$ 行加到倒数第 $m$ 行上，可得：

$$
\begin{vmatrix}
  m & 0 & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  0 & m & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & m & -1 & -1 & \cdots & -1 \\
  0 & 0 & \cdots & 0 & 1 & 1 & \cdots & 1 \\
  0 & 0 & \cdots & 0 & -\frac{n - 1}{m} & n - \frac{n - 1}{m} & \cdots & -\frac{n - 1}{m} \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & 0 & -\frac{n - 1}{m} & -\frac{n - 1}{m} & \cdots & n - \frac{n - 1}{m} \\
\end{vmatrix}
$$

再将倒数第 $m$ 行乘 $\frac{n - 1}{m}$ 后加在最后 $m - 1$ 行上，可得：

$$
\begin{vmatrix}
  m & 0 & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  0 & m & \cdots & 0 & -1 & -1 & \cdots & -1 \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & m & -1 & -1 & \cdots & -1 \\
  0 & 0 & \cdots & 0 & 1 & 1 & \cdots & 1 \\
  0 & 0 & \cdots & 0 & 0 & n & \cdots & 0 \\
  \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & 0 & 0 & 0 & \cdots & n \\
\end{vmatrix}
$$

显然上式等于 $n^{m - 1}m^{n - 1}$，得证．

那么这题的答案就是 $\binom{n - 1}{k - 1}k^{n - k - 1}(n - k)^{k - 1}$．

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
const int N = 5e5;

int n, k, P;

i64 fpow(i64 b, i64 p) {
	i64 res = 1;
	for(; p; b = b * b % P, p >>= 1) {
		if(p & 1) res = res * b % P;
	}
	return res;
}

i64 fac[N + 5], ifac[N + 5];
void pre() {
	fac[0] = 1;
	for(int i = 1; i <= N; i++) fac[i] = fac[i - 1] * i % P;
	ifac[N] = fpow(fac[N], P - 2);
	for(int i = N - 1; i >= 0; i--) ifac[i] = ifac[i + 1] * (i + 1) % P;
}
i64 C(int n, int m) { return fac[n] * ifac[m] % P * ifac[n - m] % P; }

int main() {
	n = rd(), k = rd(), P = rd(); pre();
	printf("%lld\n", fpow(k, n - k - 1) * fpow(n - k, k - 1) % P * C(n - 1, k - 1) % P);
	return 0;
}
```

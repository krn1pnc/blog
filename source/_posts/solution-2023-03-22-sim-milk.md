---
title: 2023.3.22 省选模拟赛 毒奶 (milk) 题解
date: 2023-03-22 23:31:53
categories: 题解
tags:
  - 图论
  - 动态规划
  - 容斥
  - 状态压缩
---

## 题面

### 题目描述

现实世界中有 $n$ 个城市，编号为 $1 \sim n$，城市之间有道路相连，道路是双向的，小C 可以从道路的任意一端走到另一端．因为交通并不发达，所以一些城市之间可能无法互相到达．

梦境和现实是非常相似的，所以幻想世界中也有 $n$ 个城市，编号为 $1^\prime \sim n^\prime$，还有一些双向道路．

小 C 相信梦境和现实并没有本质的区别．她选择了一种方法，把梦境中的城市和现实中的城市一一对应起来，从而在两个世界对应的城市之间移动．具体地，如果现实世界的城市 $a$ 对应幻想世界的城市 $b^\prime$，那么小 C 可以从 $a$ 走到 $b^\prime$ 或从 $b^\prime$ 走到 $a$．

因为幻想世界的交通也并不发达，所以两个世界中的道路共有恰好 $n - 1$ 条．

这样一来，小 C 周游世界的可行性就取决于对应方法的选取．定义一种对应方法是合法的，当且仅当这种对应方法使小 C 能够周游世界．

小 C 想知道，有多少种对应方法是合法的．两种对应方法不同，当且仅当存在一个现实世界的城市，在两种方法中对应不同的幻想世界的城市．

因为答案可能非常大，所以你只需要告诉她答案模 $998244353$ 的值．

### 数据范围

$3 \le n \le 20$，$0 \le m \le n - 1$．

### 时间限制

10.0 s．

## 思路

题目中给定的点有 $2n$ 个，边有 $n - 1$ 条，显然联通块总个数不等于 $n + 1$ 则无解．只需要对联通块之间连边方案计数．

设 $f_S$ 代表 $S$ 内联通块互相联通的方案数．由于 dp 是对联通块考虑，有 $|S| \le n + 1$，可以状态压缩储存 $S$．

考虑满足何种条件的联通块之间才能连边．发现连边过程是一个匹配过程，只有当两端联通块的大小相同时，才能连接这两个联通块．

预处理 $C_S$、$D_S$ 代表现实 / 梦境 $S$ 中联通块的大小总和．记 $P$ 为现实中的联通块构成的集合，$Q$ 为梦境中的联通块构成的集合，我们称一个集合 $S$ 是合法的，当且仅当 $C_{S \cap P} = D_{S \cap Q}$．

那么对于一个合法的集合 $S$，所有匹配方案数显然是 $C_{S \cap P}!$．但是其中包含了一些不合法的匹配方案，需要我们将其容斥掉．

考虑如何去掉不能使 $S$ 内联通块互相联通的匹配方式．对于 $S$ 的一个合法的真子集 $T$，钦定 $T$ 和 $S \setminus T$ 中的点分别连边，这样做的方案数为 $C_{T \cap P}! \times f_{S \setminus T}$．

枚举所有合法的真子集 $T$，我们有如下转移方程：
$$
f_S = C_{S \cap P}! - \sum_T C_{T \cap P}! \times f_{S \setminus T}
$$
其中 $S \setminus T$ 表示 $T$ 关于 $S$ 的补集．

计算联通块大小可以使用并查集；枚举真子集可以钦定 $S$ 中编号最小的联通块不被选，然后枚举子集即可．

## 代码

```cpp
#include <cstdint>
#include <cstdio>

using i64 = int64_t;
constexpr int N = 23;
constexpr i64 P = 998244353;

int n, m;
i64 fac[N + 5];

int fa[N + 5], siz[N + 5];
int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }

int ct1[1 << N], ct2[1 << N];

int calc(int m, int *sz) {
	for (int i = 1; i <= n; i++) fa[i] = i, siz[i] = 1;
	for (int i = 1, u, v; i <= m; i++) {
		scanf("%d%d", &u, &v);
		int fu = find(u), fv = find(v);
		if (fu != fv) fa[fv] = fu, siz[fu] += siz[fv];
	}
	int psiz[N + 5], ct = 0;
	for (int i = 1; i <= n; i++) {
		if (find(i) == i) psiz[ct++] = siz[i];
	}
	for (int S = 0; S < (1 << ct); S++) {
		for (int i = 0; i < ct; i++) {
			if (S & (1 << i)) sz[S] += psiz[i];
		}
	}
	return ct;
}

i64 f[1 << N];

int main() {
	scanf("%d%d", &n, &m);

	fac[0] = 1;
	for (int i = 1; i <= n; i++) fac[i] = fac[i - 1] * i % P;

	int n1 = calc(m, ct1), n2 = calc(n - 1 - m, ct2);

	int tot = n1 + n2;
	if (tot != n + 1) return puts("0"), 0;

	int msk = (1 << n1) - 1;
	for (int S = 0; S < (1 << tot); S++) {
		if (ct1[S & msk] != ct2[S >> n1]) continue;
		f[S] = fac[ct1[S & msk]];
		for (int U = S ^ (S & (-S)), T = U; T; T = (T - 1) & U) {
			if (ct1[T & msk] != ct2[T >> n1]) continue;
			f[S] = (f[S] - fac[ct1[T & msk]] * f[S ^ T] % P + P) % P;
		}
	}

	printf("%lld\n", f[(1 << tot) - 1]);

	return 0;
}
```

---
title: 2023.3.31 省选模拟赛 基因改造 (reform) 题解
date: 2023-04-06 19:42:30
categories: 题解
tags:
  - 字符串
  - 字符串哈希
---

## 题面

### 题目描述

TB 正走在改造人类智慧基因的路上．

TB 发现人类智慧基因一点也不萌萌哒，导致人类普遍智商过低，为了拯救低智商人群，博爱的 TB 开始改造人类智慧基因．

人类智慧 DNA 由 $C$ 种人类智慧脱氧核苷酸构成（这是一种十分特殊的 DNA）．

TB 智慧 DNA 片段 $T$ 长度为 $m$，她可以把另一段长度为 $m$ 的人类智慧 DNA 片段 $S$ 改造成 $T$．

改造过程中，TB 可以充分发挥智慧，将任意两种人类智慧脱氧核苷酸交换（比如对于片段 $S = \texttt{12321}$，交换 $\texttt{1}$ 和 $\texttt{2}$ 变成 $S = \texttt{21312}$，交换 $\texttt{1}$ 和 $\texttt{4}$ 变成 $\texttt{42324}$），可以无限次交换。如果 $S$ 可以通过若干次交换变成 $T$，那么就称 $S$ 为“萌萌哒人类基因片段”．

TB 想知道对于一个长度为 $N$ 的人类智慧 DNA 片段 $S[1 \sim m]$，有多少个长度为 $m$ 的连续子片段 $S[i \sim i + m - 1]$，是“萌萌哒人类基因片段”，并且这些“萌萌哒人类基因片段”在哪里．

### 数据范围

$n,m,C \le 10^6$．

### 时空限制

2.0 s，256 MB．

## 思路

~~省选之后还补模拟赛题是否搞错了什么？~~

考虑如何判断序列 $A$ 能否改造出 $B$．

对于一个序列 $A$，记 $p(x)$ 为 $A_x$ 上次出现的位置．特别地，若 $x$ 是 $A_x$ 第一次出现的位置，则 $p(x) = 0$．

定义一个序列 $A$ 的本质序列为 $A^\prime$，其中 ${A^\prime}_x = x - p(x)$．容易发现对于两个序列 $A$，$B$，若 $A^\prime$ 和 $B^\prime$ 完全相同，那么序列 $A$ 可以改造出序列 $B$．

那么我们可以对 $T^\prime$ 进行字符串哈希，枚举 $S$ 所有长度为 $m$ 的子区间进行判断．但是重算 $S$ 子段的哈希代价是 $O(m)$ 的，无法通过本题．发现我们可以从左到右扫描 $S$ 的子区间，这样我们只需要考虑一次移动对字符串哈希的贡献就行了．

容易发现一次移动只会删除一个数再加入一个数．删除数 $S_x$ 时，若区间内 $S_x$ 出现过第二次，就扣掉对应位置的哈希值；加入数 $S_x$ 时，若 $S_x$ 在当前区间中出现过了，则加上对应的哈希值．区间整体右移时，若采用常用的哈希方式，只用将哈希值除以你选取的底数．注意除法在模意义下进行，需要逆元．

时间复杂度 $O(n)$．

## 代码

```cpp
#include <cstdio>
#include <cstring>

using i64 = long long;

constexpr int N = 1e6;
constexpr i64 P = 998244353;
constexpr i64 B = 19260817;
constexpr i64 iB = 494863259;

int T, tot;
int n, m;
int a[N + 5], b[N + 5];
int ap[N + 5];

int ans[N + 5], tp;
int pa[N + 5], sa[N + 5];

i64 pw[N + 5], ipw[N + 5];
void pre() {
	pw[0] = ipw[0] = 1;
	for (int i = 1; i <= N; i++) {
		pw[i] = pw[i - 1] * B % P;
		ipw[i] = ipw[i - 1] * iB % P;
	}
}

int main() {
	scanf("%d%d", &T, &tot), pre();
	while (T--) {
		memset(pa, 0, sizeof(pa));
		memset(sa, 0, sizeof(sa));

		scanf("%d%d", &n, &m);
		for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
		for (int i = 1; i <= m; i++) scanf("%d", &b[i]);

		for (int i = 1; i <= n; i++) ap[a[i]] = 0;
		for (int i = 1; i <= n; i++) {
			if (ap[a[i]]) pa[i] = ap[a[i]];
			ap[a[i]] = i;
		}
		for (int i = 1; i <= n; i++) ap[a[i]] = 0;
		for (int i = n; i >= 1; i--) {
			if (ap[a[i]]) sa[i] = ap[a[i]];
			ap[a[i]] = i;
		}

		i64 t = 0;
		for (int i = 1; i <= m; i++) ap[b[i]] = 0;
		for (int i = 1; i <= m; i++) {
			if (ap[b[i]]) {
				(t += pw[i] * (i - ap[b[i]]) % P) %= P;
			}
			ap[b[i]] = i;
		}
		i64 s = 0;
		for (int i = 1; i <= m; i++) {
			if (pa[i]) (s += pw[i] * (i - pa[i]) % P) %= P;
		}

		tp = 0;
		for (int i = 1; i + m - 1 <= n; i++) {
			if (s == t) ans[++tp] = i;
			if (i + m - 1 == n) break;
			if (sa[i] && sa[i] <= i + m - 1) {
				(s += P - pw[sa[i] - i + 1] * (sa[i] - i) % P) %= P;
			}
			s = (s * iB) % P;
			if (i < pa[i + m]) {
				(s += pw[m] * (i + m - pa[i + m]) % P) %= P;
			}
		}

		printf("%d\n", tp);
		for (int i = 1; i <= tp; i++) {
			printf("%d%c", ans[i], " \n"[i == tp]);
		}
	}
	return 0;
}
```

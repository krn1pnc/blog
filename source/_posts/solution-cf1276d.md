---
title: Codeforces 1276D Tree Elimination 题解
date: 2023-07-18 00:13:43
categories: 题解
tags:
  - 动态规划
---

## 思路

首先可以发现，对序列计数就是对擦除标记的方式计数．

对于一个点 $u$，考察 $u$ 邻域中的两个点 $a, b$，我们记 $a \prec b$ 表示边 $(u, a)$ 先于边 $(u, b)$ 出现，$a \succ b$ 表示边 $(u, a)$ 后于边 $(u, b)$ 出现．

进行一个 DP．设 $f_{u, 0 / 1 / 2 / 3}$ 表示考虑了 $u$ 子树，$u$ 没有被擦除 / $u$ 被 $\prec f$ 的儿子擦除 / $u$ 被 $f$ 擦除 / $u$ 被 $\succ f$ 的儿子擦除，其中 $f$ 代表点 $u$ 的父亲．

分 $4$ 类转移：

1. 考虑转移 $0$ 情况．这时 $u$ 的儿子必定都被擦除，且儿子 $v$ 的擦除时间一定不后于 $(u, v)$ 这条边被考虑．这是因为考虑 $(u, v)$ 这条边时，为了保全 $u$ 不被擦除，$v$ 一定会被擦．那么有转移

  $$
    f_{u, 0} = \prod_v (f_{v, 1} + f_{v, 2})
  $$

2. 考虑转移 $1$ 情况．我们可以从儿子中钦定一个 $\prec f$ 的 $v$ 作为擦掉 $u$ 的点．组合其他点的方案时，如果存在 $\prec v$ 的儿子 $w$ 不被擦除或者在考虑边 $(u, w)$ 之后被擦，那么 $u$ 会被 $w$ 擦而不是被 $v$ 擦，这样的状态不合法．由于在考虑 $(u, v)$ 时 $u$ 被擦了，那么 $\succ v$ 的儿子显然就不能被 $u$ 擦了．那么有转移

  $$
    f_{u, 1} = \sum_{v \prec f} (f_{v, 0} + f_{v, 3}) \times \prod_{w \prec v} (f_{w, 1} + f_{w, 2}) \times \prod_{w \succ v} (f_{w, 0} + f_{w, 1} + f_{w, 3})
  $$

3. 考虑转移 $2$ 情况．由于 $u$ 被 $f$ 擦了，那么所有 $\prec f$ 的儿子 $v$ 都不能擦 $u$，所有 $\succ f$ 的儿子 $v$ 都不能被 $u$ 擦．有转移

  $$
    f_{u, 2} = \prod_{v \prec f} (f_{v, 1} + f_{v, 2}) \times \prod_{v \succ f} (f_{v, 0} + f_{v, 1} + f_{v, 3})
  $$

4. 考虑转移 $3$ 情况．于 $1$ 情况类似，枚举 $\succ f$ 的儿子钦定擦除即可．有转移

  $$
    f_{u, 3} = \sum_{v \succ f} (f_{v, 0} + f_{v, 3}) \times \prod_{w \prec v} (f_{w, 1} + f_{w, 2}) \times \prod_{w \succ v} (f_{w, 0} + f_{w, 1} + f_{w, 3})
  $$

暴力转移是 $O(n^2)$ 的，无法通过本题．瓶颈在于 $1$，$3$ 两种状态的转移．注意到如果按照加入顺序访问出边，后面那两个 $\prod$ 可以通过预处理前后缀积弄掉．时间复杂度 $O(n)$．

## 代码

```cpp
#include <cstdio>
#include <utility>
#include <vector>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

using pii = std::pair<int, int>;

using i64 = long long;
const i64 P = 998244353;

const int N = 2e5;

int n;
std::vector<pii> g[N + 5];

int fid[N + 5];
void pre(int u, int fa) {
	for (auto [v, id] : g[u]) {
		if (v == fa) continue;
		fid[v] = id, pre(v, u);
	}
}

i64 f[N + 5][4];
void dfs(int u, int fa) {
	for (auto [v, _] : g[u]) if (v != fa) dfs(v, u);

	static int V[N + 5]; int tp = 0;
	for (auto [v, _] : g[u]) if (v != fa) V[++tp] = v;

	static i64 pre[N + 5], suf[N + 5]; pre[0] = suf[tp + 1] = 1;
	for (int i = 1; i <= tp; i++) {
		int v = V[i];
		pre[i] = (f[v][1] + f[v][2]) % P;
		suf[i] = (f[v][0] + f[v][1] + f[v][3]) % P;
	}
	for (int i = 1; i <= tp; i++) (pre[i] *= pre[i - 1]) %= P;
	for (int i = tp; i >= 1; i--) (suf[i] *= suf[i + 1]) %= P;

	f[u][0] = 1, f[u][2] = 1;
	for (int i = 1; i <= tp; i++) {
		int v = V[i];
		(f[u][0] *= (f[v][1] + f[v][2]) % P) %= P;
		if (fid[v] < fid[u]) {
			(f[u][2] *= (f[v][1] + f[v][2]) % P) %= P;
			(f[u][1] += (f[v][0] + f[v][3]) % P * pre[i - 1] % P * suf[i + 1] % P) %= P;
		} else {
			(f[u][2] *= (f[v][0] + f[v][1] + f[v][3]) % P) %= P;
			(f[u][3] += (f[v][0] + f[v][3]) % P * pre[i - 1] % P * suf[i + 1] % P) %= P;
		}
	}
}

int main() {
	n = rd();
	for (int i = 1; i <= n - 1; i++) {
		int u = rd(), v = rd();
		g[u].emplace_back(v, i);
		g[v].emplace_back(u, i);
	}
	fid[1] = n, pre(1, 0), dfs(1, 0);
	printf("%lld\n", (f[1][0] + f[1][1]) % P);
	return 0;
}
```

## 参考

qazswedx, _DP 选讲_

xht, [_Codeforces Round #606 (Div. 1, based on Technocup 2020 Elimination Round 4) 题解_](https://www.xht37.com/codeforces-round-606-div-1-based-on-technocup-2020-elimination-round-4-%e9%a2%98%e8%a7%a3/#Tree_Elimination)

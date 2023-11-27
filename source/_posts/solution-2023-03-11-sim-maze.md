---
title: 2023.3.11 省选模拟赛 谜 (maze) 题解
date: 2023-03-16 23:16:44
categories: 题解
tags:
  - 图论
  - 二分图
  - 矩阵
  - 行列式
  - bitset
---

## 题面

### 题目描述

舞池上有 $n$ 个男生与 $n$ 个女生．给定矩阵 $M=(m_{i,j})_{n \times n}$，若 $m_{i,j} = 1$，表示男生 $i$ 和女生 $j$ 可以配对．舞会组织者想知道有多少方案能使得每个人都有舞伴？注意公平起见，一个人不能有多个舞伴．

除了这 $n$ 个男生，还有 $m$ 个男备胎站在舞会边上．假如舞池中有男生身体不适，备胎中的男生可以把他暂时替换掉．

当然，除了这 $m$ 个男备胎，还有 $k$ 个女备胎，她们的作用也是类似的．

现在有 $q$ 个事件，有两类：

1. $0\ u\ v$，表示用男备胎 $v$ 替换掉男生 $u$．
2. $1\ u\ v$，表示用女备胎 $v$ 替换掉女生 $u$．

注意替换都是暂时的，过了这次事件后都会换回来．每次替换后，你都要告诉组织者合法方案数．问题是方案数可能很大，所以你只需要告诉他这个值模 $2$ 是多少就可以了．

### 数据范围

$n,m,k \le 1000$，$q \le 100000$．

## 思路

对于一个长度为 $n$ 的排列 $p$，我们让男生 $i$ 和女生 $p_i$ 匹配，容易发现这个匹配是完美匹配的充要条件是 $\displaystyle\prod_{i = 1}^{n} M_{i,p_i} = 1$．

考虑所有这样的排列就是考虑了所有匹配情况，所以所给二分图的完美匹配数目为：
$$
\sum_{p \in S_n} \prod_{i = 1}^{n} M_{i,p_i}
$$
其中 $S_n$ 为所有长度为 $n$ 的排列所构成的集合．

这是所给矩阵 $M$ 的积和式 $\operatorname{perm}(M)$．而我们知道，在这个问题中，矩阵 $M$ 的行列式 $\det(M) \equiv \operatorname{perm}(M) \pmod 2$．证明是平凡的，考虑二式差的奇偶性即可．

每次替换一个男生或女生，就是替换矩阵 $M$ 的一行或一列，所以问题转化为求原矩阵替换一行一列后的行列式．

使用 Gauss 消元求解行列式，若每次重新计算 $\det(M)$，时间复杂度 $O(q n^3)$，会 TLE．

由矩阵的 Laplace 展开可知，我们若先处理原矩阵的伴随矩阵 $M^*$，那么有：
$$
\det(M) = \sum_{j = 1} (-1)^{i + j} M_{i,j} {M^*}_{i,j}
$$

注意到替换某一行或某一列时候，伴随矩阵中对应的行或列的值不会改变，所以可以 $O(n^3)$ 预处理伴随矩阵，$O(n)$ 查询．

当 $\det(M) \not= 0$ 时，$M$ 可逆，直接使用 $M^* = \det(M)M^{-1}$ 计算．

当 $\det(M) = 0$ 时，$M$ 不可逆．设 $M$ 的秩为 $r$，那么 $M$ 一定与 $J$ 等价，其中 $J$ 代表 $r$ 阶单位矩阵．也就是说存在可逆矩阵 $P,Q$，使得 $PMQ = J$．那么有：
$$
M^* = \left( P^{-1} J Q^{-1} \right)^* = \frac{Q J^* P}{\det(P) \det(Q)}
$$

当 $r < n - 1$ 时，$J^*$ 为全 $0$ 矩阵；当 $r = n - 1$ 时，${J^*}_{n,n} = 1$，其他的元素为 $0$；当 $r = n$ 时，$J^* = J$．

然而 $n \le 1000$，预处理都跑不完．发现矩阵 $M$ 是 $01$ 矩阵，那么相关运算都可以使用 `bitset` 优化．

最终时间复杂度 $O(n^3 / w + qn / w)$．

## 代码

```cpp
#include <bitset>
#include <cstdio>

constexpr int N = 1e3;

int n;

using vec = std::bitset<N + 5>;
struct mat {
	vec _m[N + 5];

	vec &operator[](int i) { return _m[i]; }
	const vec &operator[](int i) const { return _m[i]; }

	static mat I() {
		mat R;
		for (int i = 1; i <= n; i++) {
			R[i][i] = 1;
		}
		return R;
	}

	mat T() const {
		mat R;
		for (int i = 1; i <= n; i++) {
			for (int j = 1; j <= n; j++) {
				R[i][j] = _m[j][i];
			}
		}
		return R;
	}

	friend mat operator*(const mat &A, const mat &B) {
		mat T = B.T(), R;
		for (int i = 1; i <= n; i++) {
			for (int j = 1; j <= n; j++) {
				R[i][j] = (A[i] & T[j]).count() & 1;
			}
		}
		return R;
	}
} A;

int gauss(mat &A, mat &R) {
	R = mat::I();
	int r = 0;
	for (int i = 1, p = 1; i <= n; i++) {
		if (!A[p][i]) {
			for (int j = p + 1; j <= n; j++) {
				if (A[j][i]) {
					std::swap(A[p], A[j]);
					std::swap(R[p], R[j]);
					break;
				}
			}
		}
		if (!A[p][i]) continue;
		for (int j = 1; j <= n; j++) {
			if (j != p && A[j][i]) {
				A[j] ^= A[p], R[j] ^= R[p];
			}
		}
		p += 1, r += 1;
	}
	return r;
}

mat adj, adjt;
void getadj(mat A) {
	mat P, Q;
	int r = gauss(A, P);
	A = A.T(), gauss(A, Q), Q = Q.T();

	if (r < n - 1) return;
	else if (r == n - 1) adj[n][n] = 1;
	else adj = mat::I();

	adj = Q * adj * P;
	adjt = adj.T();
}

int m, k, q;
vec chi[N + 5], chj[N + 5];

char buf[N + 5];
int main() {
	scanf("%d", &n);
	for (int i = 1; i <= n; i++) {
		scanf("%s", buf + 1);
		for (int j = 1; j <= n; j++) {
			A[i][j] = buf[j] - '0';
		}
	}
	scanf("%d%d", &m, &k);
	for (int i = 1; i <= m; i++) {
		scanf("%s", buf + 1);
		for (int j = 1; j <= n; j++) {
			chi[i][j] = buf[j] - '0';
		}
	}
	for (int i = 1; i <= k; i++) {
		scanf("%s", buf + 1);
		for (int j = 1; j <= n; j++) {
			chj[i][j] = buf[j] - '0';
		}
	}

	getadj(A);
	printf("%d\n", (int)(adjt[1] & A[1]).count() & 1);

	scanf("%d", &q);
	for (int i = 1, opt, u, v; i <= q; i++) {
		scanf("%d%d%d", &opt, &u, &v);
		if (!opt) printf("%d\n", (int)(adjt[u] & chi[v]).count() & 1);
		else printf("%d\n", (int)(adj[u] & chj[v]).count() & 1);
	}

	return 0;
}
```

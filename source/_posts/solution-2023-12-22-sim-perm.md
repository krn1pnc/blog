---
title: 2023.12.22 省选模拟赛 排列 (perm) 题解
date: 2023-12-22 20:40:15
categories: 题解
tags:
  - 图论
---

## 题面

【数据删除】

## 思路

第一眼：什么 b 限制．

先考虑如何判断一个排列合法．不容易想到的是，对于排列的所有区间，我们将区间内的二元组看成一条边，显然会连成一个 DAG，而限制就是在说，这个 DAG 上若存在 $a \to b$ 和 $b \to c$ 的边，也一定存在 $a \to c$ 的边，即具有传递性．

事实上由于每条边只会出现一次，只需要每个前缀和后缀构造出的 DAG 具有传递性即可．

令边全集 $U = \{(u, v) \mid 1 \le u < v \le n\}$，考虑 $G = (V, E)$ 和 $G^\prime = (V, U - E)$ 都有传递性的 $E$ 的数量．打个暴搜发现恰好是 $n!$，怎么回事呢？

> 引理：记满足条件的 $E$ 构成的集合为 $A$，存在 $S_n \to A$ 的双射，其中 $S_n$ 表示全体 $n$ 阶排列构成的集合．

证明：

直接构造 $f(p) = \{(p_i, p_j) \mid i < j, p_i < p_j\}$，有 $U - f(p) = \{(p_i, p_j) \mid i > j, p_i < p_j\}$，显然 $f(p) \in A$．

显然这样构造出的 $f$ 是单射，考虑证明 $f$ 是满射．延续上述构造思路，建图后 $1$ 没有入度，将与其有连边的点编号全丢到 $1$ 右边，与其没有连边的点编号全丢到 $1$ 左边，然后左右变成子问题，递归构造即可．显然上述过程会构造出一个 $p \in S_n$．

得证．

事实上证明中给出的构造还有更好的性质：令 $f(p) = E$，若向 $E$ 中新增一条边 $(u, v)$ 后仍然满足条件，那么存在 $1 \le i < n$，使得 $p_i = u, p_{i + 1} = v$．故可以将 $p$ 看做点，若 $E^\prime = f(p) \cup \{(u, v)\}$ 合法，建边 $(p, f^{-1}(E^\prime))$，显然也是个 DAG，且每个点的出边只有 $O(n)$ 条！原二元组排列计数问题转化为 DAG 上路径计数．

现在考虑加上限制，对于一条限制 $(a, b), (c, d)$，若 $(c, d)$ 比 $(a, b)$ 先加入边集就寄了，故状态 $p$ 不合法的判定条件是，若令 $p_x = a, p_y = b, p_z = c, p_w = d$，有 $x > y \land z < w$，搜索的过程中判一下就好．

时间复杂度 $O(n! \times n)$．

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

const int N = 30;
const int NF = 3628800;
const i64 P = 998244353;

int fac[N + 5];

int n, m;
int a[N + 5], b[N + 5], c[N + 5], d[N + 5];
i64 f[NF];

bool fl[N + 5];
int p[N + 5], ip[N + 5], suf[N + 5];
void dfs(int x) {
    if (x == n + 1) {
        for (int i = 1; i <= m; i++) if (ip[c[i]] > ip[d[i]] && ip[a[i]] < ip[b[i]]) return;
        int u = 0; for (int i = 1; i <= n; i++) u += fac[n - i] * suf[i];
        f[u] = !u;
        for (int i = 1; i <= n - 1; i++) {
            if (p[i] > p[i + 1]) {
                int v = u + fac[n - i] * (suf[i + 1] - suf[i]) + fac[n - (i + 1)] * (suf[i] - 1 - suf[i + 1]);
                (f[u] += f[v]) %= P;
            }
        }
        return;
    }
    int ct = 0;
    for (int i = 1; i <= n; i++) {
        if (fl[i]) continue;
        p[x] = i, ip[i] = x, suf[x] = ct, fl[i] = true;
        dfs(x + 1);
        p[x] = ip[i] = 0, ct++, fl[i] = false;
    }
}

int main() {
    n = rd(), m = rd();
    for (int i = 1; i <= m; i++) {
        a[i] = rd(), b[i] = rd(), c[i] = rd(), d[i] = rd();
    }
    fac[0] = 1; for (int i = 1; i <= n; i++) fac[i] = fac[i - 1] * i;
    dfs(1);
    printf("%lld\n", f[fac[n] - 1]);
    return 0;
}
```

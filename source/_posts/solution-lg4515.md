---
title: 洛谷 4515 [COCI2009-2010#6] XOR 题解
date: 2024-01-05 08:00:56
categories: 题解
tags:
  - 数学
  - 容斥
  - 动态规划
---

## 思路

记第 $i$ 个三角形的左下顶点坐标为 $(a_i, b_i)$，腰长为 $c_i$．以下分析省略三角形面积中 $\frac{1}{2}$ 的常数，最后一起除！

直接考虑容斥，凑一下可以发现系数是 $(-2)^{|S| - 1}$．设 $f_S$ 表示集合 $S$ 中三角形交的面积，答案为：

$$
\sum_{S} (-2)^{|S| - 1} f_S
$$

考虑如何计算 $f_S$．画图可以发现，这些三角形的交也是一个等腰三角形，且顶点为 $(\max_{i \in S} a_i, \max_{i \in S} b_i)$，斜边解析式为 $y = -x + \min_{i \in S} \{a_i + b_i + c_i\}$．故面积为 $(\min_{i \in S} \{a_i + b_i + c_i\} - \max_{i \in S} a_i - \max_{i \in S} b_i)^2$．

此时直接暴力枚举集合计算，复杂度 $O(n 2^n)$，足以通过本题．但是我们考虑一个更牛的做法就是，我们把三角形按 $a_i + b_i + c_i$ 降序排序，这样第一个 $\min$ 就只和集合中编号最大的元素有关了！于是可以设 $f_{i, j, p, q}$ 为当前考虑了前 $i$ 个三角形，选了 $j$ 个，$\max x_i = p$，$\max y_i = q$ 的方案数，有转移：

$$
\begin{aligned}
f_{i - 1, j, p, q} &\to f_{i, j, p, q} \\
f_{i - 1, j - 1, p, q} &\to f_{i, j, \max\{p, a_i\}, \max\{q, b_i\}}
\end{aligned}
$$

令 $g_{i, j, p, q}$ 为钦定 $i$ 选时的 $f_{i, j, p, q}$，答案可写作：

$$
\sum_{i = 1}^n \sum_{j = 1}^i (-2)^{j - 1} \sum_p \sum_q g_{i, j, p, q} \times \max\{0, a_i + b_i + c_i - p - q\}^2
$$

直接做，时间复杂度 $O(n^4)$．还能不能再牛一点？

考虑将容斥系数拆成 $(-1)^{|S| - 1} 2^{|S| - 1}$，然后将第二部分塞到 DP 里计算，这样就只用记录选取个数的奇偶性．具体地，我们重新设 $f_{i, b, p, q}$ 为考虑了前 $i$ 个三角形，选了 $j$ 个，$j \bmod 2 = b$，$\max x_i = p$，$\max y_i = q$ 的方案数乘 $2^{j - 1}$ 的值，有转移：

$$
\begin{aligned}
f_{i - 1, b, p, q} &\to f_{i, b, p, q} \\
2 \times f_{i - 1, (b - 1) \bmod 2, p, q} &\to f_{i, b, \max\{p, a_i\}, \max\{q, b_i\}}
\end{aligned}
$$

对 $g_{i, j, p, q}$ 做类似的改动，答案可写作：

$$
\sum_{i = 1}^n \sum_{j < 2} (-1)^{j + 1} \sum_p \sum_q g_{i, j, p, q} \times \max\{0, a_i + b_i + c_i - p - q\}^2
$$

时间复杂度 $O(n^3)$．

实现上有个细节就是，中间结果可能很大，但是答案是在 $10^6 \times 10^6 = 10^{12}$ 这个范围之内的，故可找个大于这个范围的数，对其取模．

## 代码

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
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

using i128 = __int128_t;

const int N = 100;
const i128 P = 1000000000000000003;
const i128 I2 = 500000000000000002;

int n; struct ele { int x, y, l; } a[N + 5];
int bx[N + 5], xc, by[N + 5], yc;
i128 pw[N + 5], f[2][2][N + 5][N + 5];

int main() {
    n = rd();
    for (int i = 1; i <= n; i++) {
        int x = rd(), y = rd(), l = rd();
        a[i] = {x, y, l}, bx[i] = x, by[i] = y;
    }
    std::sort(bx + 1, bx + 1 + n), xc = std::unique(bx + 1, bx + 1 + n) - (bx + 1);
    std::sort(by + 1, by + 1 + n), yc = std::unique(by + 1, by + 1 + n) - (by + 1);
    std::sort(a + 1, a + 1 + n, [](ele a, ele b) { return a.x + a.y + a.l > b.x + b.y + b.l; });
    for (int i = 1; i <= n; i++) a[i].x = std::lower_bound(bx + 1, bx + 1 + xc, a[i].x) - bx;
    for (int i = 1; i <= n; i++) a[i].y = std::lower_bound(by + 1, by + 1 + yc, a[i].y) - by;

    i128 ans = 0;

    f[0][0][0][0] = I2;
    for (int i = 1; i <= n; i++) {
        memset(f[i & 1], 0, sizeof(f[i & 1]));
        static i128 g[2][N + 5][N + 5]; memset(g, 0, sizeof(g));
        for (int j = 0; j <= 1; j++) {
            for (int p = 0; p <= xc; p++) {
                int np = std::max(p, a[i].x);
                for (int q = 0; q <= yc; q++) {
                    int nq = std::max(q, a[i].y);
                    (f[i & 1][j][p][q] += f[(i - 1) & 1][j][p][q]) %= P;
                    (f[i & 1][j][np][nq] += 2 * f[(i - 1) & 1][j ^ 1][p][q]) %= P;
                    (g[j][np][nq] += 2 * f[(i - 1) & 1][j ^ 1][p][q]) %= P;
                }
            }
        }
        for (int j = 0; j <= 1; j++) {
            i128 tot = 0, L = bx[a[i].x] + by[a[i].y] + a[i].l;
            for (int p = 0; p <= xc; p++) {
                for (int q = 0; q <= yc; q++) {
                    i128 l = std::max<i128>(0, L - bx[p] - by[q]);
                    (tot += l * l % P * g[j][p][q] % P) %= P;
                }
            }
            (ans += (!j) ? P - tot : tot) %= P;
        }
    }

    printf("%lld.%d\n", (i64)ans >> 1, (ans & 1) ? 5 : 0);
    return 0;
}
```

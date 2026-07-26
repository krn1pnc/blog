---
title: 2026 多校 杂题选记
date: 2026-07-26 13:23:17
categories: 做题记录
---

吃到大份了．

## [牛客多校 3 I](https://ac.nowcoder.com/acm/contest/133878/I)

发现相邻交换和包含 $1, n$ 的交换的 $\Delta$ 形式复杂，所以先将形如 $(1, i), (i, n), (i, i + 1)$ 的交换贡献到答案上．然后考虑 $1 < j < i < n$ 且 $i - j > 1$ 的交换 $(i, j)$ 的贡献．

考察交换后序列价值和原序列价值的差 $\Delta$．记 $l_i = a_{i - 1}, r_i = a_{i + 1}, o_i = |l_i - a_i| + |r_i - a_i|$，有

$$
\Delta = |l_i - a_j| + |r_i - a_j| + |l_j - a_i| + |r_j - a_i| - o_i - o_j
$$

考虑枚举绝对值的符号，有

$$
\Delta = \max_{s_1, s_2, s_3, s_4 \in \{-1, 1\}} [s_1(l_i - a_j) + s_2(r_i - a_j) + s_3(l_j - a_i) + s_4(r_j - a_i) - o_i - o_j]
$$

分离 $i$ 和 $j$，有

$$
\Delta = \max_{s_1, s_2, s_3, s_4 \in \{-1, 1\}} [(s_1 l_i + s_2 r_i - s_3 a_i - s_4 a_i - o_i) + (s_3 l_j + s_4 r_j - s_1 a_j - s_2 a_j - o_j)]
$$

先枚举 $i$，考虑如何计算最大的 $\Delta$．有

$$
\begin{aligned}
\Delta_{\mathrm{max}}
&= \max_{1 < j < i - 1} \{\max_{s_1, s_2, s_3, s_4 \in \{-1, 1\}} [(s_1 l_i + s_2 r_i - s_3 a_i - s_4 a_i - o_i) + (s_3 l_j + s_4 r_j - s_1 a_j - s_2 a_j - o_j)]\} \\
&= \max_{s_1, s_2, s_3, s_4 \in \{-1, 1\}} \{\max_{1 < j < i - 1} [(s_1 l_i + s_2 r_i - s_3 a_i - s_4 a_i - o_i) + (s_3 l_j + s_4 r_j - s_1 a_j - s_2 a_j - o_j)]\} \\
&= \max_{s_1, s_2, s_3, s_4 \in \{-1, 1\}} [(s_1 l_i + s_2 r_i - s_3 a_i - s_4 a_i - o_i) + \max_{1 < j < i - 1} (s_3 l_j + s_4 r_j - s_1 a_j - s_2 a_j - o_j)]
\end{aligned}
$$

枚举 $s_1, s_2, s_3, s_4$ 的所有可能取值，对每个取值预处理第二项即可快速计算．

## [牛客多校 3 J](https://ac.nowcoder.com/acm/contest/133878/J)

我操我真想不到这个．

考察如何确定一个点 $u$ 的父亲．若 $u$ 没有任何被要求的祖先，直接连到 $1$ 上显然是最优的．考察 $u$ 的祖先约束集合 $\mathrm{anc}_u$，一个简单的想法是直接连到 $\argmax_{v \in \mathrm{anc}_u} \mathrm{dep}_v$ 上．不妨设该策略选择的点为 $p_u$，若存在 $w$ 满足 $u \in \mathrm{anc}_w$，且存在 $x \in \mathrm{anc}_w, x \not \in \mathrm{anc}_u, \mathrm{dep}_{p_u} < \mathrm{dep}_x < \mathrm{dep}_u$，问题就出现了：$u$ 的父亲至少应该是 $x$，不然 $w$ 的祖先约束无法被全部满足．

这启发我们按照深度降序确定节点在新树中的父亲．对于叶子节点 $u$，此问题不存在，直接选择 $\mathrm{anc}_u$ 中深度最深的点 $v$．选择后，为使 $u$ 的其他祖先约束被满足，这些点必须成为 $v$ 的祖先，即将 $\mathrm{anc}_u$ 中除 $v$ 外的点加入 $\mathrm{anc}_v$．写一个启发式合并堆模拟该过程即可．
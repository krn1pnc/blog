---
title: Lagrange 插值复习笔记
date: 2023-07-25 19:48:41
categories: 学习笔记
tags:
  - Lagrange 插值
---

Lagrange 插值是一种使用 $n + 1$ 个点值确定出一个 $n$ 次多项式的算法．

设我们的点值为 $(x_i, y_i)$，需要确定的多项式为 $f(x)$．考虑对每个 $i$ 构造系数 $k_i$ 满足 $k_i = [x = x_i]$，显然 $\sum_i k_iy_i$ 是满足要求的多项式．

让 $k_i$ 在 $x \not= x_i$ 时为 $0$ 是容易的，令 $k_i$ 中包含因子 $\prod_{j \not= i} (x - x_j)$ 即可．如果令 $k_i = \prod_{j \not= i} (x - x_j)$，那么当 $x = x_i$ 时，$k_i$ 就是 $\prod_{j \not= i} (x_i - x_j)$，而我们这时需要 $k_i = 1$，直接除去这个因子即可．

那么我们可以得到 Lagrange 插值公式：

$$
f(x) = \sum_i y_i \prod_{j \not= i} \frac{x - x_j}{x_i - x_j}
$$

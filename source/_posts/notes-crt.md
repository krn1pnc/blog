---
title: CRT 复习笔记
date: 2023-07-23 17:22:15
categories: 学习笔记
tags:
  - CRT
---

CRT 是用来求解形如下列同余方程组的算法：

$$
\left\{
\begin{matrix}
  x & \equiv & a_1 \pmod{b_1} \\
  x & \equiv & a_2 \pmod{b_2} \\
  & \vdots & \\
  x & \equiv & a_n \pmod{b_n}
\end{matrix}
\right.
$$

其中 $b_i$ 互质．

考虑构造一组 $r_i$，使得 $r_i \bmod b_j = [i = j]$，这样 $\sum\limits_{i = 1}^n r_ia_i$ 显然是一个合法解．

构造很简单，记 $x = \prod_{j \not= i} b_j$，$xy \equiv 1 \pmod{b_i}$，有 $r_i = xy$．正确性显然．

---
title: EXGCD 复习笔记
date: 2023-07-20 18:43:49
categories: 学习笔记
tags:
  - EXGCD
---

EXGCD 是用于求解形如 $ax + by = \gcd(a, b)$ 的不定方程的算法．

考虑求解 GCD 的过程，我们容易得到 $bx^\prime + (a \bmod b)y^\prime = \gcd(b, a \bmod b) = \gcd(a, b) = ax + by$．

我们有

$$
\begin{aligned}
ax + by
&= bx^\prime + (a \bmod b)y^\prime \\
&= bx^\prime + (a - \lfloor a / b\rfloor) y^\prime \\
&= ay^\prime + b(x^\prime - \lfloor a / b \rfloor y^\prime)
\end{aligned}
$$

比对系数可得，$x = y^\prime, y = (x^\prime - \lfloor a / b \rfloor y^\prime)$．然后递归计算 $bx^\prime + (a \bmod b)y^\prime = \gcd(b, a \bmod b)$ 这个子问题．系数每次至少减半，那么算法的时间复杂度就是 $O(\log a)$．

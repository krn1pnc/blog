---
title: 树上邻域相关方程组的消元技巧
date: 2023-05-31 22:42:27
categories: Tricks
---

对于树上随机游走类题目，常见的解法是列方程组消元．暴力 Gauss 消元的复杂度不优秀，通常需要我们找出高效的递推算法．

对于一个方程组，如果它满足以下条件，那么其中所有元可以在 $O(n)$ 的时间复杂度内被推出：

1. 方程组的每个元一一对应树上某个节点．
2. 树上的某个节点对应的元都和它邻域中的点对应的元有关．

设节点 $u$ 对应的元为 $x_u$，这些方程组通常具有以下形式：

$$
x_u = c_u + \sum_{v \in N_u} k_{u, v}x_v
$$

其中 $N_u$ 指的是 $u$ 的邻域，$c_u$ 是只和 $u$ 有关的量，$k_{u, v}$ 是只和 $(u, v)$ 这条边有关的量．

首先我们随便钦定一个根，记 $p$ 是 $u$ 的父亲．然后我们抛出这样一个结论：$x_u$ 其实只和 $x_p$ 有关．

感性理解这个结论．对于我们钦定的根和叶子，这个结论显然成立．对于其他的节点，考虑从叶子往上归纳，发现除了父亲的元都是已知项，容易得到结论成立．

那么，不妨设 $x_u = a_u x_p + b_u$．记 $C_u$ 表示钦定根后 $u$ 的儿子构成的集合，尝试确定 $a_u$ 和 $b_u$：

$$
\begin{aligned}
  x_u &= c_u + k_{u, p}x_p + \sum_{v \in C_u} k_{u, v}x_v \\
  x_u &= c_u + k_{u, p}x_p + \sum_{v \in C_u} k_{u, v}(a_v x_u + b_v) \\
  x_u &= c_u + k_{u, p}x_p + x_u \sum_{v \in C_u} k_{u, v}a_v + \sum_{v \in C_u} k_{u, v}b_v \\
  \left(1 - \sum_{v \in C_u} k_{u, v}a_v\right) x_u &= c_u + k_{u, p}x_p + \sum_{v \in C_u} k_{u, v}b_v \\
  x_u &= \frac{k_{u, p}}{1 - \sum\limits_{v \in C_u} k_{u, v}a_v} x_p + \frac{c_u + \sum\limits_{v \in C_u} k_{u, v}b_v}{1 - \sum\limits_{v \in C_u} k_{u, v}a_v}
\end{aligned}
$$

观察形式可以得到：

$$
a_u = \frac{k_{u, p}}{1 - \sum\limits_{v \in C_u} k_{u, v}a_v}, b_u = \frac{c_u + \sum\limits_{v \in C_u} k_{u, v}b_v}{1 - \sum\limits_{v \in C_u} k_{u, v}a_v}
$$

那么一次递推求出 $a_u, b_u$，再次递推求出 $x_u$ 即可．

注意根对应的元和某些钦定了的元的取值．

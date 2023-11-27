---
title: 容斥原理与反演学习笔记
date: 2023-09-26 20:06:16
categories: 学习笔记
tags:
  - 容斥原理
  - 反演
---

## 容斥原理

省流：小学数学．

来看一道水题！给你拥有 A，B，C，AB，BC，AC，ABC 属性的物体个数，求总物品数．

依照经验，加上有一个属性的，减去有两个属性的，再加上有三个属性的就是答案．容易验证这是正确的！

尝试扩展一下．我们现在有属性全集 $U$，给你拥有集合 $S$ 中属性的物品数量 $f_S$，求总物品数．

按图索骥，我们可以猜测答案是

$$
\sum_{S \subseteq U} (-1)^{|S| + 1} f_S
$$

容易验证这是对的．我们空降了一个 $(-1)^{|S| + 1}$，然后和式就给出了正确的值，看起来很神奇！为啥对来着．

考察一个属性集合为 $S$ 的物品，它会被所有 $f_T, T \subseteq S$ 统计到一遍，那么总共就会被计算

$$
\sum_{i = 1}^{|S|} \binom{|S|}{i}
$$

次．现在我们希望给每个 $f_S$ 一个容斥系数 $c_{|S|}$，使得每种物品只会被计算一次，即

$$
\forall 1 \le x \le |U|, \sum_{i = 1}^x \binom{x}{i} c_i = 1
$$

此时带入 $(-1)^{|S| + 1}$ 的正确性就可以通过二项式定理轻易得出了！

可惜地，并不是所有问题的容斥系数的形式都这样简单．所以，我们需要了解一些魔术．

## 反演

### 本质

反演指的是改变表示的方向，即 $f$ 表示 $g$ $\to$ $g$ 表示 $f$．

假设我们已知系数 $a_{i, j}$，使得

$$
g_n = \sum_{i = 1}^n a_{n, i} f_i
$$

且能够通过系数 $a_{i, j}$ 反推出一组系数 $b_{i, j}$，使得

$$
f_n = \sum_{i = 1}^n b_{n, i} g_i
$$

就能知 $g$ 求 $f$，反之亦然．这一过程称作反演，上述两式互为反演公式．

可以发现，上文提到的容斥系数的构造也是在求解一组互相表示关系．这两者间存在着紧密关联．

### 第一反演公式

感觉上面都在说批话啊，我不知道咋求系数是不是还是一分不会．

反演牛逼的地方在于 $b$ 只和 $a$ 有关，而和 $f$ 和 $g$ 无关．也就是说如果我们对于某对平凡的 $f, g$ 弄出了一组系数 $a, b$，那么对于任意数列 $p, q$ 都有：

$$
p_n = \sum_{i = 1}^n a_{n, i} q_i \iff q_n = \sum_{i = 1}^n b_{n, i} p_i
$$

这看起来很反直觉，为啥一组系数可以对一车有互相表示关系的数列对成立呢．

设 $\mathbf{A} = (a_{i, j}), \mathbf{B} = (b_{i, j})$，显然这是两个下三角矩阵．假设这一互相表示关系对数列 $p, q$ 成立，将其写成列向量 $\mathbf{p}, \mathbf{q}$，那么上述的关系就能够写成

$$
\mathbf{Ap} = \mathbf{q} \iff \mathbf{Bq} = \mathbf{p}
$$

将右侧带入左侧，有 $\mathbf{ABq} = \mathbf{q}$，即 $\mathbf{AB} = \mathbf{I}$．由于 $\mathbf{A}$ 和 $\mathbf{B}$ 恒互逆，不随 $p, q$ 改变，这说明了只要我们找到一组符合条件的 $a, b, p, q$，就能将 $p, q$ 推广．

这同时也给出了一种通用的求容斥系数的方案：把矩阵写出来后直接求逆．但是大部分时候需要求解的范围是 $n \le 10^6$ 甚至 $n \le 10^9$，直接求逆显然寄飞了．可喜的是前人求出过很多系数对，可以直接用！

不过求逆做法可以拿来打表，还是有点用的！

## 一些经典的反演 / 容斥系数

大致按照笔者使用的频率排序．

### 二项式反演

$$
\begin{aligned}
  f_n = \sum_{i} \binom{n}{i} g_i &\iff g_n = \sum_{i} \binom{n}{i} (-1)^{n - i} f_i \\
  f_n = \sum_{i} \binom{i}{n} g_i &\iff g_n = \sum_{i} \binom{i}{n} (-1)^{i - n} f_i
\end{aligned}
$$

容易看出，下面那个形式本质上是将上面那个形式的系数矩阵转置得到的．

带入 $f_i = x^i, g_i = (x - 1)^i$ 容易验证正确性．

### 子集反演

$$
\begin{aligned}
  f_S = \sum_{T \subseteq S} g_T &\iff g_S = \sum_{T \subseteq S} (-1)^{|S| - |T|} f_T \\
  f_S = \sum_{S \subseteq T} g_T &\iff g_S = \sum_{S \subseteq T} (-1)^{|T| - |S|} f_T
\end{aligned}
$$

就是上面说的那个容斥．

可以扩展到可重集，让重复元素的贡献为 $0$ 即可．

### Mobius 反演

$$
\begin{aligned}
  f_n = \sum_{d \mid n} g_d &\iff g_n = \sum_{d \mid n} \mu(n / d) f_d \\
  f_n = \sum_{n \mid d} g_d &\iff g_n = \sum_{n \mid d} \mu(d / n) f_d
\end{aligned}
$$

带入 $f_i = 1, g_i = [i = 1]$ 容易通过 Dirichlet 卷积验证正确性．

可以看作对质因子构成的可重集做子集反演．

### Min-Max 容斥

### 单位根反演

### Stirling 反演

## 参考

《具体数学》5.1, 6.1

Alex_Wei, [_组合数学相关_](https://www.cnblogs.com/alex-wei/p/Combinatorial_Mathematics.html)

Alex_Wei, [_反演与狄利克雷卷积_](https://www.cnblogs.com/alex-wei/p/Dirichlet.html)

VFleaKing, [_炫酷反演魔术_](https://vfleaking.blog.uoj.ac/slide/87)

command_block, [_炫酷反演魔术_](https://www.luogu.com.cn/blog/command-block/xuan-ku-fan-yan-mo-shu)

RenaMoe, [_容斥原理学习笔记_](https://renamoe.gitee.io/2021/04/08/容斥原理学习笔记/)

RenaMoe, [_笔记 各类反演_](https://renamoe.gitee.io/2021/03/11/笔记-各类反演/)

Deadecho, [_容斥原理，容斥系数_](https://www.cnblogs.com/gzy-cjoier/p/9686787.html)

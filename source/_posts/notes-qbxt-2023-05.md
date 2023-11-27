---
title: QBXT 2023 五一集训学习笔记
date: 2023-04-29 21:48:59
categories: 学习笔记
tags:
  - 数学
  - 数论
  - 组合数学
  - 生成函数
  - 群论
  - 博弈论
---

## Day 1

都听得不是很懂啊．

### Part 1 数论

#### 剩余类环 $\Z / n\Z$

简单定义：元素集合 $\{\overline{0}, \overline{1}, \dots, \overline{n - 1}\}$，其中 $\overline{a} = \{x \mid x = a + kn, k \in \Z\}$

运算：$+$，$\times$．由环的性质存在交换律, 结合律, 分配律．

#### Fermat 小定理

> 对于任意的质数 $p$ 和整数 $0 < a < p$，有 $a^{p - 1} \equiv 1 \pmod p$．

记集合 $S = \{\overline{1}, \overline{2}, \dots, \overline{n - 1}\}$，则函数 $f: S \rightarrow S, f(\overline{x}) = \overline{ax}$ 是一个双射．

证明是容易的．首先 $f$ 显然是一个满射．考虑 $\overline{d} = \overline{ax} - \overline{ay} = \overline{a(x - y)}$，由于 $a$ 和 $x - y$ 均与 $p$ 互质（在剩余系中），那么 $d$ 也与 $p$ 互质，也就是说 $f$ 值域中的元素两两不同，这样就证明了 $f$ 同时是一个单射．

于是我们有：

$$
\prod\limits_{i = 1}^{p - 1} i \equiv \prod\limits_{x = 1}^{p - 1} ai \pmod p
$$

整理可得 $(p - 1)! \equiv a^{p - 1} \times (p - 1)! \pmod p$，即 $a^{p - 1} \equiv 1 \pmod p$．

#### Euler $\phi$ 函数

定义 $\phi(n) = \sum\limits_{i = 1}^n [\gcd(i, n) = 1]$．

即小于等于 $n$ 的正整数中与 $n$ 互质的数的个数．

#### Euler 定理

> 对于任意正整数 $p,p \ge 2$ 和 $a, \gcd(a, p) = 1$，有 $a^{\phi(p)} \equiv 1 \pmod p$．

证明与 Fermat 小定理类似，此处略去．

#### Euler $\phi$ 函数的性质

> $\phi(p^k) = p^k - p^{k - 1}$，其中 $p$ 是一质数，$k$ 是一正整数．

考虑 $1 \sim p^k$ 这 $p^k$ 个数，其中显然只有 $p$ 的倍数与 $p^k$ 不互质．去掉即得 $\phi(n)$．

> 若 $\gcd(a, b) = 1$，那么有 $\phi(ab) = \phi(a)\phi(b)$（即 $\phi$ 函数是积性函数）．

证明没讲，故略去．

> 令 $n = \prod\limits_i {p_i}^{a_i}$，有 $\phi(n) = n\prod\limits_i (1 - 1 / p_i)$，其中 $p_i$ 代表第 $i$ 个质数．

由 $\phi$ 函数的积性可得：

$$
\begin{aligned}
  \phi(n)
  &= \phi\left(\prod\limits_i {p_i}^{a_i}\right) \\
  &= \prod\limits_i \phi\left({p_i}^{a_i}\right) \\
  &= \prod\limits_i {p_i}^{a_i}(1 - 1 / p_i) \\
  &= n\prod\limits_i (1 - 1 / p_i)
\end{aligned}
$$

#### 原根

对于正整数 $n$，集合 $G = \{\overline{x} \mid \gcd(x,n) = 1\}$ 若和乘法构成循环群，且 $G = \braket{\overline{g}}$，则称 $g$ 为模 $n$ 意义下的一个原根．

更严谨地，我们称满足 $a^n \equiv 1 \pmod m$ 的最小正整数 $n$ 为元素 $a$ 模 $m$ 的阶．若 $\gcd(a, m) = 1$，且 $a$ 模 $m$ 的阶为 $\phi(m)$，则称 $a$ 为模 $m$ 意义下的一个原根．

#### 原根的性质

1. 模 $n$ 意义下存在原根当且仅当 $n = 2, 4, p^k, 2p^k$．

2. 模 $n$ 意义下原根的个数为 $\phi(\phi(n))$．

没听懂．证明见 [OI Wiki](https://oi-wiki.org/math/number-theory/primitive-root)．

#### 求最小原根

由于模 $n$ 意义下原根的个数为 $\phi(\phi(n))$，我们不妨断言，原根是均匀分布的．这启发我们枚举答案然后检验．在这里，我们不加证明地抛出原根判定定理：对于一个数 $x$，我们枚举 $\phi(n)$ 的质因数 $p$，判定 $x^{\phi(n) / p} \not\equiv 1 \pmod m$，若对于 $\phi(n)$ 的所有质因数均有上式成立，那么 $x$ 是 $n$ 的一个原根．由于不同质因数的个数是 $\log$ 级别的，而尝试次数期望为 $\dfrac{n}{\phi(\phi(n))}$，所以期望时间复杂度为 $O\left(\dfrac{n}{\phi(\phi(n))} \times \log^2(\phi(n))\right)$．

#### 指标

记 $g$ 为模 $m$ 意义下的原根，对 $\gcd(a, m) = 1$ 的 $a$，若 $a = g^u$，则记 $I(a) = u$ 为 $a$ 的指标．

发现这玩意和对数很像，类比对数的运算，我们得到 $I(ab) = I(a) + I(b)$，$I(a^k) = kI(a)$．

#### BSGS 算法

~~北上广深算法．~~

给定 $m$ 和原根 $g$，求解 $a$ 的指标．

令 $B = \sqrt{\phi(m)}$，我们预处理 $X = \left\{g^L, g^{2L}, \dots, g^{L^2}\right\}$，$Y = \left\{g, g^2, \dots, g^L\right\}$．由于 $g^k = g^{\lfloor k / B\rfloor B + k \bmod B} = \left(g^B\right)^{\lfloor k / B\rfloor} \times g^{k \bmod b}$，容易发现一定存在 $x \in X, y \in Y$，使得 $a = xy$．于是我们枚举 $x$，查找是否存在 $y \in Y$，使得 $y = x^{-1}a$ 即可．

采用 BSGS 算法可以解决底数和模数互质的离散对数问题．

#### 二次剩余

求同余方程 $x^2 \equiv n \pmod p$ 的解．只讨论 $p$ 为奇质数时的情况．

设 $g$ 为模 $p$ 下的一个原根，设 $n = g^u$，若 $u$ 是偶数则 $x = g^{u / 2}$，若 $u$ 为奇数则无解．

#### Mobius 函数

定义：

$$
\mu(n) =
\begin{cases}
  1 & n = 1 \\
  0 & n \text{ 存在平方质因子} \\
  (-1)^r & n = \prod\limits_{i = 1}^r p_i
\end{cases}
$$

> $\mu$ 函数是积性函数．

考虑两个数 $a,b$ 使得 $\gcd(a, b) = 1$，我们要证明的就是 $\mu(ab) = \mu(a)\mu(b)$．

当 $a,b$ 中有一个数存在平方质因子时，$ab$ 中也存在平方质因子，此时上式显然成立．

而 $\gcd(a,b) = 1$，若 $a,b$ 中都不存在平方质因子，则 $ab$ 中也不存在平方质因子，此时 $\mu(a)\mu(b) = (-1)^{x}(-1)^{y} = (-1)^{x + y} = \mu(ab)$．

#### Dirichlet 卷积

定义：令 $f,g$ 是两个数论函数，定义

$$
(f * g)(n) = \sum\limits_{d | n} f(d)g\left(\frac{n}{d}\right)
$$

称 $f * g$ 为 $f$ 和 $g$ 的 Dirichlet 卷积．

性质：容易验证 Dirichlet 卷积存在以下性质

1. 交换律：$f * g = g * f$

2. 结合律：$f * (g * h) = (f * g) * h$

3. 分配律：$f * (g + h) = f * g + h * g$

4. 若 $f$ 和 $g$ 是积性函数，则 $f * g$ 也是积性函数．

#### Mobius 反演

> 定义数论函数 $I_0(n) = 1$，若 $g = f * I_0$，那么 $f = g * \mu$．

设数论函数 $\Delta(n) = [n = 1]$，显然有 $f * \Delta = f$（即 $\Delta$ 是 Dirichlet 卷积的单位元）．

注意到 $(\mu * I_0)(n) = \sum\limits_{d | n} \mu(d) = \sum\limits_{d | n^\prime} \mu(d) = \sum\limits_{i = 0}^k \binom{k}{i}(-1)^i = (1 + (-1))^k = [n = 1] = \Delta(n)$．

那么对于 $g = f * I_0$，两边同时卷上 $\mu$，得到 $f = g * \mu$．

### Part 2 组合数学

#### 生成函数

定义：对于无穷数列 $\{a_0, a_1, \dots\}$，定义 $A(x) = \sum\limits_{i=0}^\infty a_i x^i$ 为这个数列的生成函数．

虽然我们一般不研究这个级数的敛散性，但为了让其收敛，我们一般认为 $x \in (0, 1)$．

记 $[x^n]F(x)$ 代表 $F(x)$ 中 $x^n$ 项前面的系数．

#### 生成函数解决组合问题

> 共有 $m$ 种物品, 每种物品有任意多个,求取 $n$ 个物品的方案数．

从组合角度来说，这个问题等价于 $n$ 个相同小球放在 $m$ 个不同盒子中，使用插板法，答案为 $\binom{n + m - 1}{m - 1}$．

考虑单个物品选 $i$ 个的方案数的生成函数，显然为 $\sum\limits_{i = 0}^\infty x^i = \dfrac{1}{1 - x}$．

然后把每个物品的生成函数乘起来，答案为 $[x^n]\dfrac{1}{(1 - x)^m}$．

由广义二项式定理可得：

$$
[x^n]\frac{1}{(1 - x)^m} = [x^n](1 - x)^{-m} = [x^n]\sum\limits_{i = 0}^\infty \binom{-m}{i}(-x)^i = [x^n]\sum\limits_{i = 0}^\infty \binom{m + i - 1}{m - 1} x^i = \binom{m + n - 1}{m - 1}
$$

和我们采用组合方法得到的答案一致．

#### 生成函数求解数列通项

有点麻烦，先咕着．

#### Catalan 数

> 求长度为 $2n$ 的括号序列的个数．

有一个组合做法，在一个 $n \times n$ 的网格图上走路，从 $(0, 0)$ 到 $(n, n)$，每一步只能向上或者向右，向上代表左括号，向右代表右括号．

显然在任意时刻左括号数大于等于右括号数，所以我们的路径不能越过 $y = x$．如果不考虑限制的话，走法有 $\binom{2n}{n}$ 种．对于任意一条不合法的路径，我们找到第一次到达 $y = x + 1$ 的地方，将之前的路径沿 $y = x + 1$ 翻折．显然一定会把起点翻折到 $(-1, 1)$．也就是说，$(-1, 1)$ 到 $(n, n)$ 的任意一条路径都和一条不合法路径一一对应．那么最终方案数为：$\binom{2n}{n} - \binom{2n}{n - 1} = \binom{2n}{n} / (n + 1)$．

我们考虑使用生成函数去求解 Catalan 数的通项．记 $h_n$ 表示第 $n$ 个 Catalan 数，$H(x) = \sum\limits_{i = 0}^\infty H_i x^i$ 是它的生成函数．

考虑 Catalan 数的递推，我们括号序列中的第一个字符肯定是左括号，枚举右括号的位置，问题就变成了括号内和括号外两个子问题．不难得到递推式：

$$
h_0 = h_1 = 1, h_n = \sum\limits_{i + j = n - 1} b_i b_j
$$

显然右边是个卷积的形式，于是我们有：$H(x) = 1 + xH(x) \times H(x) = 1 + xH^2(x)$．

解得 $H(x) = \dfrac{1 \pm \sqrt{1 - 4x}}{2x}$．

考虑 $x \rightarrow 0^+$，我们需要让级数收敛以求得 $h_0$，显然取 $+$ 号时不符合需求，那么我们就得到了 Catalan 数生成函数的封闭形式：

$$
H(x) = \frac{1 - \sqrt{1 - 4x}}{2x}
$$

为求得 Catalan 数的通项，我们有：

$$
\begin{aligned}
  h_n
  &= [x^n]H(x) \\
  &= [x^n]\frac{1 - \sqrt{1 - 4x}}{2x} \\
  &= [x^n]\frac{1 - (1 - 4x)^{1 / 2}}{2x} \\
\end{aligned}
$$

由广义二项式定理可得：

$$
\begin{aligned}
  h_n
  &= [x^n]\frac{1}{2x}\left(1 - \sum\limits_{i = 0}^\infty \binom{1 / 2}{i}(-4x)^i\right) \\
  &= [x^n]\frac{1}{2x}\left(-\sum\limits_{i = 1}^\infty \binom{1 / 2}{i}(-4x)^i\right) \\
  &= [x^n]2\sum\limits_{i = 1}^\infty \binom{1 / 2}{i}(-4x)^{i - 1} \\
  &= [x^n]2\sum\limits_{i = 1}^\infty \frac{(-4x)^{i - 1}}{i!} \prod\limits_{j = 0}^{i - 1} \left(\frac{1}{2} - j\right) \\
  &= [x^n]\sum\limits_{i = 1}^\infty \frac{(4x)^{i - 1}}{i!} \prod\limits_{j = 1}^{i - 1} \left(j - \frac{1}{2}\right) \\
  &= [x^n]\sum\limits_{i = 1}^\infty \frac{(2x)^{i - 1}}{i!} \prod\limits_{j = 1}^{i - 1} \left(2j - 1\right) \\
  &= [x^n]\sum\limits_{i = 1}^\infty \frac{x^{i - 1}}{i!} \prod\limits_{j = 1}^{i - 1} 2(2j - 1) \\
  &= [x^n]\sum\limits_{i = 1}^\infty \frac{x^{i - 1}}{i!} \prod\limits_{j = 1}^{i - 1} \frac{2j - 2}{j - 1} (2j - 1) \\
  &= [x^n]\sum\limits_{i = 1}^\infty \frac{(2i - 2)!}{i!(i - 1)!} x^{n - 1} \\
  &= [x^n]\sum\limits_{i = 0}^\infty \frac{(2i)!}{i!(i + 1)!} x^n \\
  &= [x^n]\sum\limits_{i = 0}^\infty \frac{1}{i + 1} \binom{2i}{i} x^n \\
  &= \frac{1}{n + 1} \binom{2n}{n}
\end{aligned}
$$

与组合方法得到的式子一致．

#### 第一类 Stirling 数

记 $n \brack k$ 为第一类 Stirling 数，表示将 $n$ 个不同的元素分成 $k$ 个互不区分的圆排列的方案数．

> 递推公式：${n \brack k} = {n - 1 \brack k - 1} + (n - 1){n - 1 \brack k}$

考虑加入一个元素时的情况：

- 如果加入的元素自己构成一个新的圆排列，方案数为 $n \brack k$．
- 如果加入的元素插入到已有的一个圆排列中，方案数为 $(n - 1){n - 1 \brack k}$．

> 上升幂转普通幂：$[x^k]x^{\overline{n}} = {n \brack k}$

考虑第 $n$ 行 Stirling 数的生成函数 $f_n(x) = \sum\limits_{i = 0}^\infty {n \brack i} x^i$．

由递推公式可得，$f_n(x) = xf_{n - 1}(x) + (n - 1)f_{n - 1}(x) = (x + n - 1)f_{n - 1}(x)$，又有边界条件 $f_1(x) = x$，所以 $f_n(x) = x^{\overline{n}}$，即 $[x^k]x^{\overline{n}} = {n \brack k}$．

#### 第二类 Stirling 数

记 $n \brace k$ 为第二类 Stirling 数，表示将 $n$ 个不同的元素分成 $k$ 个互不区分的集合的方案数．

> 递推公式：${n \brace k} = {n - 1 \brace k - 1} + k{n - 1 \brace k}$

考虑加入一个数的情况：

- 如果加入的元素自己构成一个新的集合，方案数为 $n - 1 \brace k - 1$．
- 如果加入的元素插入到一个已有的集合中，方案数为 $k{n - 1 \brace k}$．

> 普通幂转下降幂：$x^n = \sum\limits_{i = 0}^n {n \brace i}x^{\underline{i}}$

左侧相当于将 $n$ 个数分成 $x$ 个不同的集合的方案数，而右边相当于枚举集合数并逐一计算贡献．

### Part 3 群论

只听懂了 $1 / 3$，记点结论好了．

#### 陪集定理

设 $(G, \times)$ 构成一个群，$H \le G$ 是 $G$ 的一个子群，那么 $H$ 的所有不同左 / 右陪集构成了 $G$ 上的一个划分．

#### 轨道稳定集定理

对于一个群 $G$ 和一个集合 $A$，

记 $E_a = \{b \mid b = g \circ a, g \in G\}$，称 $E_a$ 为 $a$ 的轨道．

记 $Z_a = \{g \in G \mid g \circ a = a\}$，称 $Z_a$ 为 $a$ 的稳定集．

有：$|E_a||Z_a| = |G|$．

#### Burnside 引理

对于一个群 $G$ 和一个集合 $A$，有 $G$ 在 $A$ 上的一个群作用 $\circ$，Burnside 引理指出，轨道的个数 $C$ 满足下面的式子：

$$
C = \frac{1}{|G|} \sum\limits_{g \in G} \sum\limits_{a \in A} [g \circ a = a]
$$

## Day 2

终于来点我会的了！

### Part 1 线性代数

前面之前就会，故略过．

#### Cauchy–Binet 定理

设 $A$ 是 $m \times n$ 矩阵，$B$ 是 $n \times m$ 矩阵．

对于一个大小为 $m$ 的指标集合 $S$，设 $A_S$ 表示 $A$ 中列指标位于 $S$ 中的列构成的矩阵，$B_S$ 表示 $B$ 中行指标在 $S$ 中的行构成的矩阵，Cauchy–Binet 定理告诉我们：

$$
\det(AB) = \sum\limits_{S} \det(A_S)\det(B_S)
$$

证明参考 [Wikipedia](https://en.wikipedia.org/wiki/Cauchy-Binet_formula)．

#### 矩阵树定理

上课没听懂，本节内容主要抄自《数学天书中的证明》．

设有一无向图 $G = (V, E)$．考虑 $G$ 的关联矩阵 $B = (b_{i, e})$，其行的指标集为 $V$，列的指标集为 $E$，有 $b_{i, e} = [i \in e]$．

将 $B$ 每一列中的两个 $1$ 中的任意一个换成 $-1$（相当于给 $G$ 定向），记得到的矩阵为 $C$．则 $M = CC^T$ 是对称的 $n \times n$ 矩阵，且主对角线上是顶点的度数．

> 矩阵树定理：去掉 $M$ 中的第 $i$ 行和第 $i$ 列（$i = 1, \dots, n$），记得到的矩阵为 $M^\prime$，$\det(M^\prime)$ 为图 $G$ 的生成树个数．

对 $M^\prime$ 应用 Cauchy-Binet 定理，我们得到：

$$
\det(M^\prime) = \sum\limits_N \det(N)\det(N^T) = \sum\limits_N (\det(N))^2
$$

其中 $N$ 取遍 $C$ 去掉第 $i$ 行后的所有 $(n - 1) \times (n - 1)$ 子矩阵．显然 $N$ 对应图 $G$ 中 $n - 1$ 条边构成的一个子图，故只需证明

$$
\det(N) =
\begin{cases}
  \pm 1 & \text{当这些边生成树} \\
  0 & \text{Otherwise}
\end{cases}
$$

若这 $n - 1$ 条边不生成树，那么 $N$ 可以通过初等行变换使得某一列全部为 $0$，故此时 $\det(N) = 0$．

当这 $n - 1$ 条边生成树时，我们考虑一个删点序列 $j_1, \dots, j_{n - 1}$，使得每次按顺序删除点 $j_k$ 时均有 $j_k$ 的度数为 $1$，且对于任意的 $k$ 有 $j_k \not = i$．记删除点 $j_k$ 时移除的那条边为 $e_k$．现在置换行使得 $j_i$ 在第 $i$ 行，$e_i$ 在第 $i$ 列，由构造可知对于 $a < b$，有 $j_a \not\in e_b$，所以新的矩阵 $N^\prime$ 是下三角的，且对角线元素为 $\pm 1$，即 $\det(N) = \pm\det(N^\prime) = \pm 1$．得证．

### Part 2 多项式科技基础

#### FFT

先咕着．

#### 分治 FFT

先咕着．

## Day 3

### Part 1 概率论

略过．

### Part 2 博弈论

它 来 了．

#### 组合博弈

组合博弈一般来说分为两种：平等博弈和不平等博弈．

- 平等博弈指可允许的操作只和当前局面的状态有关而和操作的玩家无关的博弈．
- 不平等博弈中可允许的操作则还和当前操作玩家相关．

一般我们研究的游戏都是平等博弈游戏．

#### $\text{NP}$ 图

我们按照如下方式定义 $\text{N}$ 态和 $\text{P}$ 态：

1. 结束状态（即无法进行移动的状态）是 $\text{P}$ 态．
2. 可以移动到 $\text{P}$ 态的状态都是 $\text{N}$ 态．
3. 只能移动到 $\text{N}$ 态的状态是 $\text{P}$ 态．

即 $\text{N}$ 态为必胜态，$\text{P}$ 态为必败态．

将游戏的一个状态向它能移动到的所有状态连边，那么构成的图我们称作 $\text{NP}$ 图．

#### Bash 博弈

> 有 $n$ 个石子，玩家轮流取 $1 \sim x$ 个石子，取走最后一枚石子的玩家获胜．求先手何时必胜．

观察 Bash 博弈的 $\text{NP}$ 图，我们可以发现，其所有 $\text{P}$ 态为：$n \equiv 0 \pmod {x + 1}$．

那么 $n \not\equiv 0 \pmod {x + 1}$ 时，有先手必胜．

#### Nim 博弈

> 有 $n$ 堆石子，第 $i$ 堆石头有 $a_i$ 个，玩家轮流从任意一堆石子中取出至少一个，取走最后一枚石子的玩家获胜．求先手何时必胜．

经典结论：$a_i$ 异或和为 $0$ 时，先手必败．其他状态先手必胜．

如何证明？考虑证明 $\text{NP}$ 图的性质：

1. 显然结束状态有 $a_1 = \cdots = a_n = 0$，此时有异或和为 $0$，是 $\text{P}$ 态．
2. 如果当前状态异或和不为 $0$，不妨假设 $\bigoplus\limits_{i = 1}^n = k \not= 0$，且我们将 $a_i$ 变为 ${a_i}^\prime$ 后，原式异或和为 $0$．那么有 ${a_i}^\prime = a_i \oplus k$．假设 $k$ 的二进制最高位为 $t$，一定存在奇数个 $a_x$，使得 $a_x$ 的第 $t$ 位为 $1$．我们任选一个作为 $a_i$，都有 $a_i > {a_i}^\prime$．这样就证明了 $\text{N}$ 态能够移动到 $\text{P}$ 态．
3. 如果当前状态有异或和为 $0$，那么无论玩家如何选取，异或和都不会为 $0$．这样就证明了 $\text{P}$ 态只能移动到 $\text{N}$ 态．

#### Moore’s Nim

> 有 $n$ 堆石子，第 $i$ 堆石头有 $a_i$ 个，玩家轮流从不超过 $k$ 堆石子中取出至少一个，取走最后一枚石子的玩家获胜．求先手何时必胜．

结论：将每一个数写成二进制形式，如果每一位上 $1$ 的个数都是 $k + 1$ 的倍数，则先手必败，否则先手必胜．

还是考虑证明 $\text{NP}$ 图的性质：

1. 与 Nim 博弈中关于结束状态的证明一致．
2. 如果当前状态不满足结论，则玩家一定可以通过一次移动使状态满足结论．略过直接但繁琐的证明过程，我们说明了 $\text{N}$ 态能够移动到 $\text{P}$ 态．
3. 如果当前状态满足结论，则不存在一个合法移动使得移动后的状态也满足结论．假设我们对一个满足结论的状态操作后，其仍然满足结论，如果我们是通过操作某一位得到的，那么必定需要操作 $k + 1$ 堆石子才能得到这样的状态，而这是不合法的．这样我们就证明了 $\text{P}$ 态只能移动到 $\text{N}$ 状态．

#### Sprague-Grundy 游戏

给定 $n$ 个 DAG，每个 DAG 上有且仅有一个起点，起点上有一枚棋子，有若干个终止节点．两个玩家进行游戏，每次每个人选择一个 DAG，并将其上的棋子移动一条边．不能移动的人输．

定义一张图上一个节点 $u$ 的 $\operatorname{SG}$ 值为：

$$
\operatorname{SG}(u) = \text{mex}\left(\operatorname{SG}(v)\right)
$$

其中 $v$ 取遍所有 $u$ 的后继节点．对于终止节点 $t$ 有 $\operatorname{SG}(t) = 0$．

> 一组游戏先手必胜当且仅当所有 DAG 的初始节点的 $\operatorname{SG}$ 值的异或和不为 $0$．

证明与 Nim 博弈结论的证明类似．故略去．

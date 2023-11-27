---
title: 洛谷 5324 [BJOI2019] 删数 题解
date: 2023-05-25 14:49:06
categories: 题解
tags:
  - 线段树
---

## 思路

这个删数操作定义得十分诡异，不妨先来观察一下性质．

记 $c_x$ 表示 $x$ 在序列中出现的次数．考虑一个 $[0, n]$ 的数轴，数轴上第 $i$ 个位置的权值为 $c_i$．有一个 bot 初始在 $n$ 处，设当前位置为 $x$，它先查看自己脚下的权值 $c_x$，然后走 $c_x$ 步，到 $x - c_x$．重复这个过程，我们发现，如果我们把 bot 当前所在的位置看做当前序列的长度，走路的过程就对应删数的过程，而走 $c_x$ 步就相当于删去序列中所有等于序列长度的值，若 bot 走到了 $0$，就代表我们删完了数．

那么我们枚举出现过的所有数，设当前枚举的数为 $x$，我们将 $[x - c_x + 1, x]$ 这个区间覆盖上，表示 bot 若走到 $x$，就会走完 $[x - c_x + 1, x]$，到达 $x - c_x$．覆盖完成后，若数轴上还存在空位，那么 bot 肯定走不到终点．于是我们猜测这样操作完后 $[1, n]$ 中 $0$ 的个数就是答案．

证明：首先考虑合法性，由于序列中有 $n$ 个数，肯定存在构造方案使得数轴被完全覆盖；再考虑最优性，若序列中存在空位，bot 会在空位前停下来，也就走不到 $0$，无法删完．所以序列必须要被覆盖完，最小代价即为空位的数量．

于是我们用线段树维护这个数轴，题目中的全局 $\pm 1$ 操作可以通过懒标记干掉，我们要支持的是区间 $\pm 1$，查询 $0$ 的个数．乍一看很不可做，但是我们的 $-1$ 操作只会发生在之前 $+1$ 过的区间（相当于撤销贡献），所以不会出现负数，直接维护最小值和最小值出现次数即可．

修改时注意只有 $[1, n]$ 之内的数能造成数轴上的覆盖．

## 代码

```cpp
#include <cstdio>
#include <map>

inline int rd() {
	int x = 0, f = 1, c = getchar();
	while (((c - '0') | ('9' - c)) < 0)
		f = c != '-', c = getchar();
	while (((c - '0') | ('9' - c)) > 0)
		x = x * 10 + c - '0', c = getchar();
	return f ? x : -x;
}

const int N = 1e6;

int n, m, a[N + 5];
std::map<int, int> ct;

#define lch (p * 2)
#define rch (p * 2 + 1)
#define mid ((cl + cr) >> 1)
struct pii {
	int mn, ct;
	friend pii operator+(pii a, pii b) {
		int mn = std::min(a.mn, b.mn);
		int ct = (mn == a.mn) * a.ct + (mn == b.mn) * b.ct;
		return {mn, ct};
	}
};
struct node {
	pii v; int tg;
	void upd(int d) {
		v.mn += d, tg += d;
	}
} t[(N * 2) << 4];
void pushup(int p) { t[p].v = t[lch].v + t[rch].v; }
void pushdown(int p) {
	if(!t[p].tg) return;
	t[lch].upd(t[p].tg);
	t[rch].upd(t[p].tg);
	t[p].tg = 0;
}
void build(int p = 1, int cl = -N, int cr = N) {
	if(cl == cr) return t[p] = {0, 1, 0}, void();
	build(lch, cl, mid), build(rch, mid + 1, cr);
	pushup(p);
}
void upd(int l, int r, int d, int p = 1, int cl = -N, int cr = N) {
	if(cl == l && cr == r) return t[p].upd(d);
	pushdown(p);
	if(r <= mid) upd(l, r, d, lch, cl, mid);
	else if(l > mid) upd(l, r, d, rch, mid + 1, cr);
	else upd(l, mid, d, lch, cl, mid), upd(mid + 1, r, d, rch, mid + 1, cr);
	pushup(p);
}
int que(int l, int r, int p = 1, int cl = -N, int cr = N) {
	if(cl == l && cr == r) return !t[p].v.mn ? t[p].v.ct : 0;
	pushdown(p);
	if(r <= mid) return que(l, r, lch, cl, mid);
	else if(l > mid) return que(l, r, rch, mid + 1, cr);
	else return que(l, mid, lch, cl, mid) + que(mid + 1, r, rch, mid + 1, cr);
}
#undef lch
#undef rch
#undef mid

int main() {
	n = rd(), m = rd();
	for(int i = 1; i <= n; i++) ct[a[i] = rd()]++;
	build();
	for(int i = 1; i <= n; i++) {
		if(ct[i]) upd(i - ct[i] + 1, i, 1);
	}

	int b = 0;
	for(int i = 1; i <= m; i++) {
		int p = rd(), x = rd();
		if(!p) {
			if(x == 1) {
				if(ct[b + n]) upd(b + n - ct[b + n] + 1, b + n, -1);
				b--;
			} else {
				b++;
				if(ct[b + n]) upd(b + n - ct[b + n] + 1, b + n, 1);
			}
		} else {
			if(a[p] <= b + n) upd(a[p] - ct[a[p]] + 1, a[p] - ct[a[p]] + 1, -1);
			ct[a[p]]--; a[p] = b + x; ct[a[p]]++;
			if(a[p] <= b + n) upd(a[p] - ct[a[p]] + 1, a[p] - ct[a[p]] + 1, 1);
		}
		printf("%d\n", que(b + 1, b + n));
	}
	return 0;
}
```

## 参考

E_huan, [_P5324 题解_](https://www.luogu.com.cn/blog/Ehuan/p5324-ti-xie)

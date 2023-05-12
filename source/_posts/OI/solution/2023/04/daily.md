---
title: "2023年4月做题简要记录"
date: 2023-4-20
description: "2023年4月做题简要记录"
tags: ["daily"]
---


## [P5304 [GXOI/GZOI2019]旅行者](https://www.luogu.com.cn/problem/P5304)

给定一张有向图图，有若干个关键点，求所有关键点中距离最短的一对的距离。

$n, m \le 10^5$

每次选取两个点集 $A$ 和 $B$ ，以 $A$ 为起点，跑一边最短路，则 $\min_{b \in B} dis[B]$ 为 $\min_{a \in A, b \in B} \operatorname{dis}(a, b)$ 。

考虑目标点对 $a \not= b$ ， $a$ 中的一位二进制和 $b$ 中的一位二进制不相同，所以每次通过二进制上的某一位划分 $A$ 和 $B$ ，跑 $\log n$ 次最短路即可。

注意有向图，每次要从 $A$ 跑一次，从 $B$ 跑一次

[提交记录](https://www.luogu.com.cn/record/108696322)

## [P3248 [HNOI2016]树](https://www.luogu.com.cn/problem/P3248)

有一棵「模板树」，下称 树A 。

有一棵「大树」，下称 树B 。

最初，树B和树A是一样的。

每次操作，把树A中以某个点为根的子树复制到树B的某个点下（树A中的那个点成为新的树B的儿子），然后按在原树中的编号大小顺序重新标号。

然后若干次询问，每次询问两个点之间的距离。

询问树上的距离，常用的操作是lca+深度，这里也这么做。

考虑每次操作，形成一个新的「块」，所有的块构成了一棵树，我们维护这棵树上的深度和lca。

重新标号用主席树处理（以 $u$ 为根的子树中编号第 $i$ 小的节点，或者以 $u$ 为根的子树中 $x$ 第几小），在 $dfn$ 序列上做区间第 $k$ 小即可。

树A中的编号与树B中的编号极容易弄混，需要认真对待。

[提交记录 ~~(loj Ofast+少爷机就是好)~~](https://loj.ac/s/1759749)

$n, m \le 10^5$

## [P3267 [JLOI2016/SHOI2016]侦察守卫](https://www.luogu.com.cn/problem/P3267)

给定一棵树和若干个树上的点，每次可以在 $u$ 用 $w_u$ 的代价覆盖距离 $u$ 小于等于 $d$ 的点，求覆盖所有的关键点需要的最小代价。

对于每个点，设 $f[u][i]$ 表示还要满足还要向下覆盖 $i$ 层最小代价， $g[u][i]$ 表示可以向上覆盖 $i$ 层最小代价。

那么有，初始化

$g[u][0] = w[u]\cdot \operatorname{Important?}(u)$

$i\gt 1, g[u][i]=w[u]$

转移
$$
f[u][i] = f'[u][i] + f[v][i - 1]\\

g[u][i] = \min(g[u][i] + f[v][i], f'[u][i + 1] + g[v][i+1])\\

g[u][i] = \min_{j>i}(g[u][i], g[u][j]\\

f[u][i] = \min_{j<i}(f[u][i], f[u][j]\\

f[0] = g[0]
$$
（关于 $g[u][i] \leftarrow f'[u][i+1] + g[v][i+1]$ 的转移解释）

```
U 还能向上  D 还能向下

[u] (U i D i)
 +--------------------+          --- (Total: i+1)
 |                    |           |
[v] (U i+1)          [w] (D i-1)  |
                      .           |
                      .           |
                      .           |
                                 ---
```

[提交记录](https://www.luogu.com.cn/record/108766138)

## [P3687 [ZJOI2017]仙人掌 题解](https://www.luogu.com.cn/problem/P3687)

有一张图，问有多少种加边方案，使得加边后的图是一个仙人掌（一条边至多在一个简单环中）。

若最初的图不是仙人掌，返回 0 。

否则删去所有的环边，得到一个森林，考虑每棵树的情况。

对于每棵子树，强制选择其中的一个点可以向上连边（不连的情况就是其向它的父亲连一条重边），设为 $f[u]$ 

对于点 $u$ 考虑儿子 $v$ 中的所有向上连边的点，他们有两种方案：

1. 连向 $u$；
2. 连向其他的 $v$ 中的向上连边的点。

我们用 $h[i]$ 表示 $i$ 个点按以上方案连边的的方案数：

$$
\begin{align*}
h[i] &=& h[i-1]      &\ \ \text{这个点连向} u \\
     &+& (i-1)h[i-2] &\ \ \text{从剩下} i-1 \text{个点中选一个}
\end{align*}
$$

所以有

$$
\begin{align*}
f[u] &=& h[deg[u]] \prod_{v \in son[u]} f[v] & \ \ \text{选择} u \text{作为向上连边的点} \\
     &+& \sum_{v} h[deg[u] - 1]\ f[v] \prod_{w \in son[u], w \not= v} f[w] & \ \ \text{选择} v \text{作为向上连边的点} \\
     &=& h[deg[u] + 1] \prod_{v \in son[u]} f[v] \\
\end{align*}
$$

但是对于根，由于不能选择其作为向上连边的点（为什么？仔细想一下，其不能向上连边，向下的情况都已经计算过了）。

所以

$$
f[root] = h[deg[root]] \prod_{v \in son[root]} f[v]
$$

[提交记录](https://www.luogu.com.cn/record/108871534)

## [P3643 [APIO2016] 划艇](https://www.luogu.com.cn/problem/P3643)

有若干个篮子，每个篮子要么不放东西，要么放 $[a_i, b_i]$ 个东西，且要大于之前所有篮子放东西的个数。

$n \le 500, a_i \le b_i \le 10^9$

由于 $a_i, b_i$ 非常大，考虑离散化，形成若干个区间 $[a'_i, b'_i)$ 。

考虑 $i$ 在区间 $j$ 内选择，$k$ 是第一个在区间 $j$ 之前选择的， $k+1 \cdots i$ 都在区间 $j$ 内选择，$1 \cdots k$ 都在区间 $1 \cdots j$ 内选择

令其的方案数为 $f[i][j]$ （强制 $i$ 选择， $k \lt i$ 随意）。

那么有 $f[i][j] = \sum_{k=0}^{i-1} \sum_{l\lt j} f[k][j]\ sum(k + 1, i)$

其中 $sum(k + 1, i)$ 为从 $k+1$ 到 $i$ 中，在区间 $j$ 内选择的方案数（如果无法满足就不选， $i$ 必须选）。

令 $c = cnt(j, k+1 \cdots i)$ ，即 $k + 1\cdots i$ 中可以取到区间 $j$ 的方案数，那么 $sum(k + 1, i) = {b'[j] - a'[j] + c - 1 \choose c}$

[提交记录](https://www.luogu.com.cn/record/108889516)

## [P4139 上帝与集合的正确用法](https://www.luogu.com.cn/problem/P4139)

求 $2^{2^{2^\cdots}} \bmod p$ 

有扩展欧拉定理，对于任意 $b \ge \varphi(p)$
$$
a^b = a^{(b\ \bmod \varphi(p)) + \varphi(p)} (\bmod p)
$$
显然 $2^{2^{2^\cdots}} >> \varphi(p)$
$$
f(p) = \begin{cases} 0 & p=1 \\ 2^{f(\varphi(p)) + \varphi(p)}\ \bmod\ p & \text{otherwise} \end{cases}
$$

## [P3750 [六省联考 2017] 分手是祝愿](https://www.luogu.com.cn/problem/P3750)

给定一个 01 串，每次操作，给定一个 $i$ 翻转所有 $1 \le j \le i, j | i$ ，要让全为 $0$ 。

若当前能在 $k$ 步之内完成操作，则进行这个操作，否则随机进行一个操作，求期望。

$n \le 10^5$

首先考虑初始状态的最优策略，由于一个 $i$ 不会被 $j \lt i$ 修改，所以可以从大到小进行贪心操作，设这时的最少操作次数为 $c$ ，操作的顺序无关紧要。

考虑 dp ，令 $f[i]$ 表示当前可以最少恰好 $i$ 步完成操作的期望。

那么对于 $i \le k$ ，有  $f[i] = i$ 。

否则 $f[i] = \frac i n f[i-1] + \frac {n-i} n f[i + 1] + 1$ ，有 $\frac i n$ 的概率进行正确的操作，剩下 $\frac {n-i} n$ 的概率进行错误的操作，可以证明，至少需要再操作一次（按回来）。

对于 $f[i]$ 的转移，推式子，令 $f[i] = f[i-1]+b[i]$ :

$$
\begin{align*}
f[i] &=& \frac i n (f[i] - b[i]) + \frac {n-i} n (f[i] + b[i+1]) + 1 \\
  0  &=& -\frac i n b[i] + \frac {n-i} n b[i+1] + 1 \\
b[i] &=& \frac {(n-i)b[i+1] + n} i \\
\end{align*}
$$

[提交记录](https://www.luogu.com.cn/record/108938078)

##  [P3270 [JLOI2016] 成绩比较](https://www.luogu.com.cn/problem/P3270)

有 $n$ 个人，$m$ 门课 ，和一个人 $\mathcal{B 神}$ ，有 $k$ 个人的每门课的成绩都比B神低（小于等于），同时每门课有最高分 $U_i$ 最低分 $1$，B神的排名是 $R_i$ （有且仅有 $R_i - 1$ 个人分数大于B神），求满足以上情形的方案数。

考虑 dp 。

设 $f[i][j]$ 为考虑前 $i$ 个人，有 $j$ 个人每门课的分数都比 B 神低，那么有如下转移
$$
f[i][j] = \sum_{k = j}^{n-1} f[i-1][k] {k \choose k-j} {n-k-1 \choose R_{i}-1-(k-j)} g[i]
$$
其中 $g[i]$ 为对于第 $i$ 门课，选定有谁分数大于B神的情况下的方案数。

考虑这门课有谁的分数比 B神高，那么他就被从“比B神低”中除名了。

考虑上一次有 $k$ 个人，那么 $k \choose k-j$ 为从其中选出有多少人分数第一次比B神高，由于一共需要 $R_i - 1$ 个人，所以还要从已经比B神高的 $n-k-1$ 个人中选 $R_i - 1 - (k-j)$ 个人。

考虑怎么求 $g[i]$ ，枚举B神的分数 $j$ ，$R_i -1$ 个人分数在 $j+1 \cdots U_i$ 中，剩下 $n - R_i$ 个人分数在 $1 \cdots j$ 中：
$$
g[i] = \sum_{j=1}^{U_i} (U_i - j)^{R_i - 1}\ j^{n - R_i}
$$
因为 $U_i$ 很大，不能暴力求，发现 $g[i]$ 是一个关于 $U_i$ 的 $n$ 次（ $\sum x^n$ 为 $n+1$ 次多项式）多项式，即：
$$
G(x) = \sum_{j = 1}^x (U_i - j)^{R_i - 1}\ j^{n - R_i}
$$
用拉格朗日插值即可。

[提交记录](https://www.luogu.com.cn/record/108962529)

## [P3746 [六省联考 2017] 组合数问题](https://www.luogu.com.cn/problem/P3746)

求 $\sum_{i = 0}^\infin {nk \choose ik + r}$ ，其中 $n \le 10^9, r \lt k \le 50$ 。

也就是求 $\sum_{i\ \bmod\ k = r} {nk \choose i}$ 。

考虑二项式定理 ${nk \choose i} = [x^i](\sum_{i} {nk \choose i} x^i) = [x^i](1 + x)^{nk}$ 。

也就是要求 $\sum_{i\ \bmod\ k = r} [x^i](1 + x)^{nk}$ ，考虑使用循环卷积，把超过 $k$ 的项卷回去，即 $H(x) = \sum_{i = 0}^{k-1} \sum_{j = 0}^{k-1} [x^i]F(x)\cdot [x^j]G(x)\cdot x^{i+j \ \bmod \ k}$

对 $(1+x)^{nk}$ 使用循环卷积快速幂即可，最后答案就是求出来的 $x^r$ 的系数，时间复杂度 $\Omicron(k^2(\log nk))$

（注意当 $k = 1$ 时，初值是 $2$ ，不是 $1+x$ ）

[提交记录](https://www.luogu.com.cn/record/109000085)

## [P4099 [HEOI2013]SAO](https://www.luogu.com.cn/problem/P4099)

有 $n$ 个点，$n-1$ 个限制，对于每个限制，要求第 $i$ 个点要排在第 $j$ 个点之前或者之后，保证不存在两个点不相关（不能划分为两个点集 $A$, $B$, $\forall a \in A, b \in B, \text{无限制} (a, b)$ ），求排列方案数。

不考虑限制的顺序的话，这就是一棵树，我们考虑用树形 dp 来求。

令 $f[u][i]$ 为 $u$ 在已经处理的 $u$ 的子树内，排名为 $i$ 的方案数，每次就是要把 $f[v][k]$ 合并到 $f[u][i]$ 上。

先考虑 $v$ 要在 $u$ 之前求。

考虑 $v$ 中有 $j$ 个点要被放在 $u$ 之前，有 $j \ge k$ ，即必须让 $v$ 放在 $u$ 之前。（有一些问题，稍后讨论），

即
$$
i = 1 \cdots siz[u], j = 0 \cdots siz[v] \\

f[u][i+j] \larr f'[u][i] {i + j - 1\choose j} {siz[u]-i + siz[v]-j \choose siz[v]-j} (\sum_{k=1}^j f[v][k])
$$
其中 $siz[u]$ 为 $u$ 已经处理过的点数，$siz[v]$ 为 $v$ 的点数，${i + j - 1\choose j}$ 就是选前 $j$ 个要被放在哪些地方，$siz[u]-i + siz[v]-j \choose siz[v]-j$ 就是选后 $siz[v] - j$  个要被放在哪些地方。

最后要 $siz[u] += siz[v]$ 。

假如 $v$ 要放在 $u$ 之后，也是同理：
$$
f[u][i+j] \larr f'[u][i] {i + j - 1\choose j} {siz[u]-i + siz[v]-j \choose siz[v]-j} (\sum_{k=j+1}^{siz[v]} f[v][k])
$$
现在有一个问题，为什么这么讨论是正确的：

每次插入时，$u$ 原先的排列和 $v$ 原先的排列中的相对顺序并没有改变，所以可以保证符合题目要求，同时也可以枚举到所有的相对位置的可能性（？？？我也不太懂，感性理解一下）。

（说一个可能会疑惑的点，考虑如下的：

```
v -> u
w -> v
w -> t
```

考虑 `wtv` 和 `u`  ，这种情况是不能转移到 `wvut` 的，但是 `wvut` 是可以从 `wvt` 转移过去的）

[提交记录](https://www.luogu.com.cn/record/108997224)

##  [P1975[国家集训队]排队](https://www.luogu.com.cn/problem/P1975)

给定一个数字串，每次交换两个位置，然后求逆序对个数。

设 $gt(l, r, x)$ 为 $l \cdots r$ 中大于 $x$ 的数的个数，$lt(l,r,x)$ 同理。

考虑交换 $l \lt r$ ，那么对于在 $l$ 之前或者 $r$ 之后的位置没有影响，而中间的贡献是 $-gt(l+1,r-1, a_r) + gt(l+1,r-1,a_l) - lt(l + 1, r - 1, a[l]) + gt(l + 1, r-1, a[r]) - [a_l > a_r] + [a_r > a_l]$ 。

考虑维护 $gt$ 和 $lt$ ，用线段树套平衡树，即线段树每个节点维护一个 FHQTreap 即可。

[提交记录](https://www.luogu.com.cn/record/109008915)

##  [P3702[SDOI2017]序列计数](https://www.luogu.com.cn/problem/P3702)

求长为 $n$ ，每个数在 $1 \cdots m$ ，和为 $p$ 的倍数，且有一个数为质数的数列的方案数。

$n \le 10^9, m \le 2\times10^7,p\le100$

考虑使用模 $p$ 下的多项式，和容斥。

令 $[x^i]F_l(x)$ 为用 $1 \cdots m$ 组成长度为 $l$ ，和模 $p$ 为 $i$ 的方案数，$[x^i]G_l(x)$ 为用 $1 \cdots m$ 中的合数组成长度为 $l$ ，和模 $p$ 为 $i$ 的方案数。

那么答案就是 $[x^0]F_n(x) - [x^0]G_n(x)$ 。

考虑如何求 $F_l(x)$ 和 $G_l(x)$ 。

若 $l=1$ :

$$
i\in [0, p-1], [x^i]F_l(x) = \sum_{j \in [1, m], j\ \bmod\ p = i} 1 \\

i\in [0, p-1], [x^i]G_l(x) = \sum_{j \in [1, m], j\ \bmod\ p = i, j \not \in \mathbb P} 1
$$

否则，合并两个长度为 $a$ 和 $b$ 的数列，显然可以从两边选择任意一个数列，然后和相加即可。

考虑循环卷积，令 $l = a + b, a, b \lt l$：

$$
F_l(x) = \sum_{i \in [0, p-1]} \sum_{j \in [0, p-1]} [x^i]F_a(x)\ [x^j]F_b(x)\ x^{(i + j)\ \bmod\ p}
$$

然后用快速幂即可。

[提交记录](https://www.luogu.com.cn/record/109021662)

## [P3760 [TJOI2017] 异或和](https://www.luogu.com.cn/problem/P3760)

有一个序列 $a_i$ ，求

$$
i, j \in [1, n], i \le j,\ \operatorname{xor} (\sum a[i \cdots j])
$$

考虑计算所有可能的 $a[i \cdots j]$ 的个数，然后答案就是所有出现奇数次的数的异或和。

令 $f_x$ 为所有可能的 $a[i \cdots j]$ 中，$x$ 的出现次数。

令 $g_x$ 为前缀和 $a[1 \cdots i]$ 中 $x$ 的出现次数。

由于有 $a[i \cdots j] = a[1 \cdots j] - a[1 \cdots (j - 1)]$ ，所以有 $f_i = \sum_{j=0}^{m-i} g_jg_{i+j}$ 。( $m = a[1 \cdots n]$ )

考虑进行变换

$$
\begin{align*}
f_i      &=& \sum_{j=0}^{m-i} g_jg_{i+j} & \\
f_{m-i'} &=& \sum_{j=0}^{i'} g_jg_{j+m-i'} & i \larr m - i'\\
f_{m-i'} &=& \sum_{j=0}^{i'} g_jg'_{i'-j} & g'_i \larr g_{m-i}\\
f_{i'} &=& \sum_{j=0}^{i'} g_jg'_{i'-j} & f'_i \larr f_{m-i}\\
\end{align*}
$$

就可以卷积了。

[提交记录](https://www.luogu.com.cn/record/109026028)

##  [P3763[TJOI2017] DNA](https://www.luogu.com.cn/problem/P3763)

有两个字符串 $S$ 和 $T$ ，问 $S$ 有多少个子串更改不超过 $3$ 个字符（长度不变）能和 $T$ 相同，其中 $S$ 和 $T$ 都由 `A, T, C, G` 四个字符组成。

分别考虑四个字符：

令上文中的 $S$ 为 $S^{(0)}$ ，$T$ 为 $T^{(0)}$ 。

设当前考虑的字符是 $ch$ ，那么 $S_i = [S^{(0)}_i = ch], T_i = [T^{(0)}_i = ch]$

令 $P_i$ 为 $S$ 以 $i$ 结尾，长度 $|T|$ 的子串只考虑这个字符的情况下的匹配个数：

$$
\begin{align*}
P_i &=& \sum_{j=0}^{m-1}T_j\ S_{i-(m-1)+j} &\\
P_i &=& \sum_{j=0}^{m-1}T'_{m-1-j}\ S_{i-(m-1)+j} & T_j = T'_{m-1-j}\\
P_i &=& \sum_{k+l=i} T'_l\ S_k & l = m-1-j,\ k = i+(m-1)+j,\ 并且不用考虑范围\\
\end{align*}
$$

卷积即可。

[提交记录](https://www.luogu.com.cn/record/109031121)

## [P3734 [HAOI2017]方案数](https://www.luogu.com.cn/problem/P3734)

有一个定义在非负整数之间的 $\sube$ ，$a \land b = a \lrArr a \sube b$

考虑在一个无限大的空间中：

$x \sube x' \rArr (x, y, z) \rarr (x', y, z)$

$y \sube y' \rArr (x, y, z) \rarr (x, y', z)$

$z \sube z' \rArr (x, y, z) \rarr (x, y, z')$

其中有若干个点不能经过，求到某个点的方案数，不能经过的点数 $n \le 10000$ 。

一个套路，求不经过某些点的方案数，可以用如下的 dp 解决：

考虑一条非法的路径，枚举它第一次经过的不能经过的点。

$f[i]$ 为不经过不能经过的点到 $i$ 的方案数，其中 $g(j, i)$ 为从点 $j$ 出发，不考虑不能经过的点的方案数。

$$
f[i] = g(0, i) - \sum_{j \not= i} f[j]\ g(j, i)
$$

现在就要求 $g(j, i)$ 怎么求。

每一次移动，都是在某些位上置 $1$ ，可以发现，路径数目只与要添加的 $1$ 的个数有关。

令 $h[i][j][k]$ 为 $x, y, z$ 上分别要置 $i, j, k$ 个 $1$ 的方案数，分别考虑从三个方向转移，那么有：

$$
\begin{align*}
h[i][j][k] &= \sum_{i'=0}^{i-1} h[i'][j][k] {i \choose i'} \\
           &+ \sum_{j'=0}^{j-1} h[i][j'][k] {j \choose j'} \\
           &+ \sum_{k'=0}^{k-1} h[i][j][k'] {k \choose k'} \\
\end{align*}
$$

[提交记录](https://www.luogu.com.cn/record/109059107)

## [P3705[SDOI2017] 新生舞会](https://www.luogu.com.cn/problem/P3705)

给定 $n \times n$ 的 $a_{i, j}$ 和 $b_{i, j}$ ，选 $n$ 个位置，每行每列有且只有一个，使得 

$$
\frac {\sum_{k=1}^n a_{x_k, y_k}} {\sum_{k=1}^n b_{x_k, y_k}}
$$

最大 。

分数规划的常用套路：

$$
\begin{align*}
\frac {\sum_{k=1}^n a_{x_k, y_k}} {\sum_{k=1}^n b_{x_k, y_k}} &=& c \ge mid \\
\frac {\sum_{k=1}^n a_{x_k, y_k}} {\sum_{k=1}^n b_{x_k, y_k}} &=& (mid + \Delta) \\
\sum_{k=1}^n a_{x_k, y_k} - (mid + \Delta)\sum_{k=1}^n b_{x_k, y_k} &=& 0 \\
\sum_{k=1}^n (a_{x_k, y_k} - (mid + \Delta)b_{x_k, y_k}) &=& 0 \\
\sum_{k=1}^n (a_{x_k, y_k} - mid\cdot b_{x_k, y_k}) &=& \Delta\sum_{k=1}^n b_{x_k, y_k}
\end{align*}
$$

若存在一种方案使得 $\sum_{k=1}^n (a_{x_k, y_k} - mid\cdot b_{x_k, y_k}) \ge 0$ ，那么 $\Delta \ge 0$ ，所以能取到 $c \ge mid$  。

考虑使用最大费用流，源点向每一行连边，每一列向汇点连边，第 $i$ 行向第 $j$ 列连费用为 $a_{i,j} - mid\cdot b_{i, j}$ 的边 ，然后判断最大费用是否大于等于 $0$ 即可（容量为 $1$ ）。

[提交记录](https://www.luogu.com.cn/record/109074277)

## [P3755 [CQOI2017]老C的任务](https://www.luogu.com.cn/problem/P3755)

给定两个序列 $x_i, y_i$ ，要求支持如下操作：

1. 输入 $l, r$ ，求
   $$
   a = \frac {\sum_{l\le i\le r} (x_i - \bar x)(y_i - \bar y)} {\sum_{l\le i\le r} (x_i - \bar x)^2} \\
   $$
   其中 $\bar x$ 为 $x_l \cdots x_r$ 的平均数，定义为 $\frac {\sum_{l\le i\le r}x_i}{r-l+1}$ ，$\bar y$ 为 $y_l \cdots y_r$ 的平均数，定义为 $\frac {\sum_{l\le i\le r}y_i}{r-l+1}$

2. 输入 $l, r, s, t$ ，令 $\forall\ l\le i\le r,\ x_i \larr x_i + s,\ y_i \larr y_i + t$ 。

3. 输入 $l, r, s, t$ ，令 $\forall\ l\le i\le r,\ x_i \larr i + s,\ y_i \larr i + t$ 。



序列问题，考虑用线段树解决，先考虑操作 1，推式子：

$$
\begin{align*}
a &=& \frac {\sum_i (x_i - \bar x)(y_i - \bar y)} {\sum_i (x_i - \bar x)^2} \\
  &=& \frac {\sum_i (x_i - \bar x)(y_i - \bar y)} {\sum_i (x_i - \bar x)^2} \\
  &=& \frac {\sum_i x_iy_i - \sum_i \bar xy_i -\sum_i \bar yx_i + \sum_i \bar x\bar y} {\sum_i (x_i - \bar x)^2} \\
  &=& \frac {\sum_i x_iy_i -  \bar x\sum_iy_i -  \bar y\sum_ix_i + \bar x\bar y(r-l+1)} {\sum_i (x_i - \bar x)^2} \\  
  &=& \frac {\sum_i x_iy_i-\bar x\sum_iy_i-\bar y\sum_ix_i\bar x\bar y(r-l+1)} {\sum_i x_i^2 - 2\bar x\sum_i x_i + \bar x^2(r-l+1)} \\
\end{align*}
$$

发现，我们需要用线段树维护这些东西：$\sum x_iy_i, \sum x_i, \sum y_i, \sum x_i^2$ 。

同时我们还需要支持两个修改操作，分别是 $x_i, y_i = i$ 和 $x_i + s, y_i + t$ ，第 3 个操作可以由这两个操作组合而成。

1. $\forall\ l\le i\le r: x_i=y_i=i$

   $$
   \begin{align*}
   \sum_i {x_iy_i} = \sum_i {x_ix_i} &\larr& \sum_{i\in[l,r]} i^2 = (\sum_{i\in[1,r]} i^2) - (\sum_{i\in[1,l-1]} i^2) = \frac {r(r+1)(2r+1)-l(l-1)(2l-1)} 6 \\
   \sum_i x_i = \sum_i y_i &\larr& \frac {r(1+r) - l(l-1)} 2
   \end{align*}
   $$

2. $\forall\ l\le i \le r: x_i \larr x_i + S, y_i \larr y_i + T$

   $$
   \begin{align*}
   \sum_i{x'}_i         &\larr& \sum_i (x_i+S)        &=& (\sum_i x_i) + (r-l+1)S \\
   \sum_i{y'}_i         &\larr& \sum_i (y_i+T)        &=& (\sum_i y_i) + (r-l+1)T \\
   \sum_i{x'}_i^2       &\larr& \sum_i (x_i+S)^2      &=& \sum x_i^2 + 2S \sum x_i + (r-l+1)S^2 \\
   \sum_i{x'}_i\ {y'}_i &\larr& \sum_i (x_i+S)(y_i+T) &=& \sum x_iy_i + S \sum y_i + T \sum x_i + (r-l+1)ST \\
   \end{align*}
   $$

[提交记录](https://www.luogu.com.cn/record/109185917)

## [P3773 [CTSC2017]吉夫特](https://www.luogu.com.cn/problem/P3773)

考虑一个长度为 $n$ 的序列互不相同 $a_i$ ，要求有多少个 $a_i$ 的不降子序列 $a_{b_1}, a_{b_2} \cdots a_{b_k}, k \ge 2$ 满足：
$$
\prod_{i=2}^k {a_{b_{i-1}} \choose a_{b_i}} \bmod 2 > 0
$$
数据范围：$n \le 211986, a_i \le 233333$ 。

原式就是要求每个 ${a_{b_{i-1}} \choose a_{b_i}} \bmod 2 = 1$ 。

因为有 $\bmod 2$ 这个条件，考虑使用卢卡斯定理。
$$
{a \choose b} \bmod p = {a/p \choose b/p} {a \bmod p \choose b \bmod p} \bmod p
$$
可以发现，当 $p = 2$ 的时候每一次进行这个操作，就是把最低一位拆出来。

所以令 $a = \langle x_t, x_{t - 1}, \ldots, x_0\rangle_2, b = \langle y_t, y_{t - 1}, \ldots, y_0\rangle_2$ ，那么
$$
{a \choose b}\bmod 2 = {x_t \choose y_t} {x_{t-1} \choose y_{t-1}} \cdots {x_0 \choose y_0}\bmod 2 = 1 \\
\dArr\\
{x_t \choose y_t} = {x_{t-1} \choose y_{t-1}} = \cdots = {x_0 \choose y_0} = 1
$$
因为 $x_i, y_i \in \{0, 1\}$ ，而
$$
{0 \choose 0} = {1 \choose 0} = {1 \choose 1} = 1 \\
{0 \choose 1} = 1 \\
$$
不难发现，这就要求在二进制下 $b$ 要是 $a$ 的子集。

回到题目中，$\forall\ 2\le i \le k, a_{b_i}\sub a_{b_{i-1}}$

令 $f_i$ 表示以 $i$ 结尾的，长度大于 $2$ 的满足如上条件的的方案数，那么：
$$
j \sub i,\ f_j \larr f_j + f_i + 1
$$
要加 $1$ 是因为可以 $i$ 开始，注意可能不存在某个 $a_t = j$

每次统计答案  $ans \larr ans + a_i$

[提交记录](https://www.luogu.com.cn/record/109136855)

## [P3813 [FJOI2017]矩阵填数](https://www.luogu.com.cn/problem/P3813)

给定一个 $h \times w$ 的矩阵，每个格子能填上一个 $1\cdots m$ 的数，同时有 $n$ 个限制，每个限制给出一个子矩阵，要求子矩阵的最大值为 $v$ ，求方案数。

$h, w \le 10^4, n \le 10$

最大值为 $v$ 较难处理，因为要求至少有一个位置取到 $v$ ，考虑容斥：$用 1\cdots v 的方案数 - 用 1\cdots v-1 的方案数$ 。

$h, w$ 较大，而 $n$ 有较小，考虑对其进行离散化，得到若干个子矩阵（左上开，右下闭），每个子矩阵有最大值限制，并用状压记录下，若取到这个最大值，有哪些限制（有取到最大值）能被满足。

然后枚举子矩阵 $i$ 和已经满足的限制 $j$ ，令 $x$ 为不取最大值的方案数， $y$ 为取到（至少一个）最大值的方案数，能满足 $s$ 的限制，那么：
$$
f_{i,j} \larr f_{i-1,j} + x \\
f_{i,j\cup s} \larr f_{i-1,j} + y \\
$$
并且 $x = (v-1)^{area}, y = v^{area} - v$ ，其中 $area$ 为其的面积。

[提交记录](https://www.luogu.com.cn/record/109133643)

## [P4735 最大异或和](https://www.luogu.com.cn/problem/P4735)

给定非负整数序列 $a$ ，初始长度为 $n$ ，有若干个操作：

1. 给定 $x$ ，在序列末尾添加 $x$ ，$n \larr n+1$

2. 给定 $l, r, x$ 求
   $$
   \max_{l\le p\le r} a_p \oplus a_{p+1} \oplus \cdots \oplus a_n \oplus x
   $$

3. 

考虑操作2，可以用前缀和 $s_i$，变为：
$$
\max_{l\le p\le r} s_{p-1} \oplus s_n \oplus x = \max_{l-1\le p\le r-1} s_p \oplus s_n \oplus x
$$
$s_n \oplus x$ 知道，要求  $s_p$ 异或上它最大，这是一个传统的问题，可以用 trie 解决。

考虑怎么满足 $l-1\le p\le r-1$ 的限制：

先考虑 $p \le r-1$ ，可以采用主席树的思想，即可持久化 trie ，每次插入一个新的数，都新开一棵 trie ，询问时就在第 $r-1$ 棵 trie 上贪心即可。

再考虑 $l-1\le p$ ，对 trie 的每个节点记录最新的节点编号 $newest$ ，若 $newest\lt l-1$ ，那么显然不满足要求。

[提交记录](https://www.luogu.com.cn/record/109125655)

## [P5787 二分图 /【模板】线段树分治](https://www.luogu.com.cn/problem/P5787)

给定一张 $n$ 个节点的图，有 $k$ 个时间，有 $m$ 会在某个时间出现，在某个时间消失，求每个时间内这个图是否是二分图。



先介绍一下如何判断一个图是二分图，使用并查集：

把每个点拆成两个：$u$ 和 $u+n$ 。

每次连边 $u, v$ ，先检查若 $u$ 和 $v$ 在同一个并查集内，则不是二分图，否则合并 $(u, v+n)$ 和 $(v, u+n)$ 。



线段树分治可以处理这样的一个问题：有若干个操作，每个操作只在 $l \cdots r$ 的时间内有效，同时还有一些询问，询问某个时间点内操作的贡献。

线段树分治将时间轴放到线段树上，每进入一个节点，处理在这个节点的 $l, r$ 内会生效的操作，离开这个节点时，**撤销** 这些操作（你很难 **删除** 这些操作带来的贡献，比如对于这题的并查集），若到了叶节点，询问即可。



回到这道题，我们建出一棵线段树，然后用类似区间操作的方法，把一条边拆成若干份，挂在若干个线段树节点（这条边要在这个线段树节点对应的时间内完全生效）上。

然后询问，到每个节点把这个节点上所有的边加入并查集（因为要撤销，所以不能路径压缩，只能按秩合并），并判断是否符合要求，若否，这个节点的子树中也不可能符合要求了。否则继续递归，到叶节点时设置答案，然后每个节点离开时，按倒序撤销边的贡献即可。

[提交记录](https://www.luogu.com.cn/record/109119508)

## [P5227 [AHOI2013]连通图](https://www.luogu.com.cn/problem/P5227)

给定一个无向连通图和若干个小集合（集合大小 $c \le 4$ ），每个小集合包含一些边，对于每个集合，你需要确定将集合中的边删掉后改图是否保持联通。集合间的询问相互独立。

用线段树分治，处理出每个边不被删掉的时间段，放到线段树上。

然后dfs线段树，用并查集维护连通性，进去的时候加入，离开的时候撤销，到每个叶节点判断是否联通（ `dsu.size[dsu.get(1)] == n` ）。

[提交记录](https://www.luogu.com.cn/record/109122673)

## [P3703 [SDOI2017]树点涂色](https://www.luogu.com.cn/problem/P3703)

一棵 $n$ 个点的有根树，其中 $1$ 号点是根节点。并且每个点上的颜色不同。

定义一条路径的权值是：这条路径上的点（包括起点和终点）共有多少种不同的颜色。

进行这几种操作：

- `1 x` 表示把点 $x$ 到根节点的路径上所有的点染上一种没有用过的新颜色。
- `2 x y` 求 $x$ 到 $y$ 的路径的权值。
- `3 x` 在以 $x$ 为根的子树中选择一个点，使得这个点到根节点的路径权值最大，求最大权值。

不难发现，每种颜色都是一条从上到下的链，令 $val[x]$ 为 $x$ 到根的这条路径的权值，那么操作2就是 $val[x] + val[y] - 2\cdot val[\operatorname{lca}(x, y)] + 1$ （需要 $+1$ 是因为 $- val[lca(x,y)]$ 会把 $lca$ 所在的那段颜色扣掉），操作三就是 $u$ 子树中做 $\max val[v]$ ，可以用 dfn + 线段树维护。

考虑用 lct 维护操作1 。lct 用一棵 splay 维护一段链，这里强制用一棵 splay 维护一段颜色，发现一个节点的 $val$ 就是这个节点到根的虚边数量 $+1$ ，并且 access 操作强制把一段链到根的链变成一棵树，那么我们就可以用其的 access 操作来实现操作1 。

具体做法：

```cpp
void access(int u)
{
    for (int v = 0; u; v = u, u = tr[u].fa) {
        splay(u);
        if (tr[u].ch[1]) seg.modify_tr(findrt(tr[u].ch[1]), 1); // u->tr[u].ch[1] 变为虚边 +1
        if (v) seg.modify_tr(findrt(v), -1); // u->v 变为实边 -1
        conn(u, v, 1);
    }
}
```

时间复杂度不会证，是均摊 $O(log\ n)$ 的

[提交记录](https://www.luogu.com.cn/record/109331709)

## [P4219 [BJOI2014]大融合](https://www.luogu.com.cn/problem/P4219)

给定 $n$ 个点，有若干个操作，一种是添加一条边，另一种是问经过某条边的路径数目有多少。保证这个图任意时刻都是森林。

森林，加边，lct 没跑了。

询问操作就是两个点的虚儿子个数 $+1$ 乘起来。

注意 access 操作的时候维护虚儿子个数即可。

[提交记录](https://www.luogu.com.cn/record/109334423)

## [P3688 [ZJOI2017] 树状数组](https://www.luogu.com.cn/problem/P3688)

有一个错误的树状数组操作，单点加 $1$，区间查询（前缀和）（对 $2$ 取模），只不过代码中把修改和查询操作的下标更新的正负号写反了。有两种操作，若干次，第一种是在 $[l,r]$ 中等概率选取一个 $x$ ，单点修改，第二种是询问某个区间得到正确结果的概率是多少。



不难发现，对 $2$ 取模单点加 $1$ 就是异或，这个错误的树状数组其实实现的是查询后缀和。

那么要求的：$a_l \oplus a_{l+1} \oplus \cdots \oplus a_r$ ，实际求的： $a_{l-1}\oplus a_l \oplus \cdots \oplus a_n \oplus a_r \oplus a_{r+1} \oplus \cdots \oplus a_n = a_{l-1}\oplus \cdots \oplus a_{r-1}$ 。

二者要相等，及 $a_l \oplus \cdots \oplus a_r \oplus a_{l-1}\oplus\cdots\oplus a_{r-1} = a_{l-1}\oplus a_{r-1} = 0$  。

即所求的是 $a_{l-1} = a_{r - 1}$ 的概率。

考虑用树套树来维护 $a_l = a_r$ 的概率。

对于第一种操作 $(L, R)$ :

令 $len = R-L+1$ 。

对于所有 $l \lt L \le r \le R$ ，有 $\frac 1 {len}$ 概率翻转。

对于所有 $L\le l\le R\lt r$ ，有 $\frac 1 {len}$ 概率翻转。

对于所有 $L\le l \le r \le R$ ，有 $\frac 2 {len}$ 概率翻转。

同时需要考虑一种特殊情况，若查询时 $l = 1$ （原操作求”前缀和“时，若 $x=0$ 返回 $0$ ）：

$(a_1 \oplus \cdots \oplus a_r) \oplus (0 \oplus a_r \oplus \cdots \oplus a_n)$

即前缀和和后缀和相不相等。

对于所有 $r \lt L \le R$ ，有 $1$ 的概率翻转。

对于所有 $L \le R\lt r$ ，有 $1$ 的概率翻转。

对于所有 $L\le r \le R$ ，有 $1 - \frac 1 {len}$ 的概率翻转（有 $\frac 1 {len}$ 的几率恰巧碰到 $r$ ） 。



现在思考一下实现细节：

树套树对于每个点，用 $l, r$ 维护  $a_l = a_r$ 的概率。

有一棵线段树，对于线段树的每一个区间，都开一个线段树。对于区间修改操作，对每个完全覆盖的区间，在对应的线段树上继续操作。对于单点查询，要注意累加所有覆盖这个点的区间的线段树上的贡献。

合并两个概率的函数：$\operatorname{merge}(x, y) = xy + (1-x)(1-y)$ （有可能都翻转，也有可能都不翻转）。



[提交记录](https://loj.ac/s/1768185)

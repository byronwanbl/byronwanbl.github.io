---
title: "2023年5月做题简要记录1"
date: 2023-5-4
description: "2023年5月做题简要记录1"
tags: ["daily"]
---

## [P3761 [TJOI2017]城市](https://www.luogu.com.cn/problem/P3761)

给定一棵带权树，你可以选择断掉任意一条边，然后加上一条边权相同的新边，让城市之间距离的最大值最小。

边数 $n\le 5000$ ，时间限制 $3000\ \text{ms}$ 。



看数据范围，可以用 $\Omicron(n^2)$ 算法。

先枚举断掉哪条边，然后最大值可能是下面这些情况之一：

1. 在同一个树中：求直径即可。
2. 经过新建的边，即一个在第一个树，另一个在第二个树。

考虑情况2，即在两个树中各选一个点，使得距离树中任意点的最大值最小。

`dfs` ，考虑点 $u$ 到树中任意点的最大值，有两种情况，1是经过父亲走下来，2是在自己的子树中。

1的话从传入一个参数 $d$ ，表示从父亲传下来的最大值，2的话已经在求直径时候求出了。

从 $u$ 转移到 $v$ ，假如 $u$ 的最长链是从 $v$ 接上来的，那么就不能把 $u$ 的最长链计入 $v$ 的 $d$ 中，否则就把 $u$ 的最长链计入 $v$ 的 $d$ 中，即 $d’=w + \max(d, f[u])$ 或者 $d'= w + \max(d,g[u])$ 。

[提交记录](https://www.luogu.com.cn/record/109533683)

## [P5283 [十二省联考 2019] 异或粽子](https://www.luogu.com.cn/problem/P5283)

给定一个序列 $a$ ，要选出 $k$ 组 $(l,r)$ ，使得 $\sum_{(l, r)} a_l \oplus a_{l+1} \oplus \cdots \oplus a_r$ 最大。

不难发现可以用前缀和，令 $s_i = \sum_{j=0}^i a_i$ ，那么 $a_l \oplus a_{l+1} \oplus \cdots \oplus a_r = s_r \oplus s_{l-1}$ 。

使用可持久化trie在 $\Omicron(\log n)$ 求出 $l\le p\le r$ ，使得 $s_p \oplus x$ 最大。

我们可以这么做（虽然这么做是对的，但我不理解为什么要这么做）：

可以用一个大根堆维护，$(x, j,i,l,r)$ 表示 $x=s_j\oplus x_i$ 所有可以选出的 $j, i$，还有 $j$ 可以选的范围 $[l, r]$  。

每次取出最大的 $x$ ，计入答案，然后把 $[l, r]$ 分成两个区间 $[l, j-1], [j+1, r]$ ，在这两个区间中分别选出 $j_1, j_2$ 使得 $s_{j_1} \oplus s_i$ 和 $s_{j_2} \oplus s_i$ 最大，然后重新插入大根堆中。

[提交记录](https://www.luogu.com.cn/record/109536038)

## [P4064 [JXOI2017]加法](https://www.luogu.com.cn/problem/P4064)

给定一个序列 $A$ 和 $m$ 个区间 $[l_i, r_i]$ ，要选择 $k$ 个区间，对于每个区间执行一次区间加 $a$ （每个区间只能选择一次），最大化序列最小值。

最大化序列最小值，考虑用二分最小值，对于比这个值要小的 $A_i$ ，要选择若干个区间加 $a$ ，使之不比 $lim$ 小，那么用哪些区间呢？贪心地用右端点最远的区间覆盖，为什么？一个区间还没有被使用，说明在这个点左边并不需要这个区间，而用右端点最远的区间，有可能覆盖更多需要的 $A_i$ 。

[提交记录](https://www.luogu.com.cn/record/109547590)

## [P4454 [CQOI2018]破解D-H协议](https://www.luogu.com.cn/problem/P4454)

给定 $g, p, A, B$ ，满足 $A = g^a (\bmod p)$, $B = g^b (\bmod p)$ ，求 $g^{ab} (\bmod p)$ 。

显然求出 $a, b$ 之一即可。



考虑使用 BSGS 算法：

$b^x = n (\bmod p)$ ，已知 $b, n, p$ ，求 $x$ ：

令 $x = it-j$ ，其中 $t = \lceil\sqrt p\rceil, i \in [1, t], j \in [0, t)$ ，

那么

$$
\begin{align*}
b^x &=& n (\bmod p) \\
b^{it-j} &=& n (\bmod p) \\
(b^t)^i &=& n\cdot b^j (\bmod p) \\
\end{align*}
$$

显然预处理出所有的 $n \cdot b^j$ ，存起来，然后枚举 $i$  就行了。

代码：

```cpp
ll bsgs(ll b, int n)
{
    map<ll, int> mp;
    n %= P;
    ll t = sqrt(P) + 1;
    for (int i = 0; i < t; i++) mp[n * qpow(b, i) % P] = i;
    b = qpow(b, t);
    if (!b) return !n ? 1 : -1;
    for (int i = 1; i <= t; i++) {
        ll x = qpow(b, i);
        int j = mp.count(x) ? mp[x] : -1;
        if (~j && i * t - j >= 0) return i * t - j;
    }
    return -1;
}
```



[提交记录](https://www.luogu.com.cn/record/109552208)

## [P4065 [JXOI2017]颜色](https://www.luogu.com.cn/problem/P4065)

给定序列 $a_i$ ，问有多少种方案数使得删去若干种数字后，最后剩下来的序列非空且不断开（例如，对于 $1, 2, 3, 4, 5$ ，删去 $2, 3$ 之后，变成了 $1, \_, \_, 4, 5$ 不合法，若删去 $1$ ，变成 $2, 3, 4, 5$ 是合法方案）。



显然要在原序列中选择非空的一段，使得这段中的所有数字仅在这段中出现过。有两种解法。



解法一：

对于每个位置，赋一个随机的值 $b_i$，并且使得所有同个数字位置上的值的和为 $0$ （$\sum_{a_i=x} b_i = 0$） ，然后对于 $b_i$ 做前缀和 $s_i$，到每个位置 $i$ 统计之前有多少个 $j$ 有 $s_i = s_j$ ，加起来即可。

一种可能的赋值方案是：设 $c_x = \langle 1 \le i \le n: a_i=x \rangle$ ，那么 $1 \le j \lt |c_x|,\ b_i = \operatorname{rand}(), b_{|c_x|} = -\sum_{1\le j \lt |c_x|} b_j$ 。



解法二：

先枚举右端点，然后考虑有多少个合法的左端点。设颜色 $x$ 最小的位置为 $mn_x$ ，最大的位置为 $mx_x$ 。先考虑段中的所有颜色不能在左端点之左：枚举到颜色 $x$ 的 $mx_x$ 时，把线段树上 $mn_x+1 \cdots mx_x$ 设为 $1$ ，表示这些点不能成为左端点。再考虑段中的颜色不能再右端点之右，即若 $mx_x \gt r$ ，那么 $x$ 一定不能被选中，用栈维护所有的 $mx_x > r$ 的点，从左至右把每个的$i$ 压入栈，然后弹出所有 $mx_{a_i} \le r$ 的 $i$  ，然后此时的栈顶就是不能取到的左端点，即此时的左端点为 $l=\operatorname{top()} + 1$ ，答案为 $r-l+1-seg.\operatorname{query}(l\cdots r)$ 。



[提交记录](https://www.luogu.com.cn/record/109600934)

## [P4782 【模板】2-SAT 问题](https://www.luogu.com.cn/problem/P4782)

要解决一个 **2-SAT** 问题：有 $n$ 个布尔变量，$x_1 \cdots x_n$ ，和 $m$ 个条件，每个条件要求 $x_i=a \lor x_j = b$ ，问是否存在一种取值使得满足所有条件，求一种可能取值，或报告无解。



一个 k-SAT 问题就是有若干个布尔变量 $x_i$ ，和若干个条件，每个条件形如 $x_{a_1}=b_1 \lor \cdots \lor x_{a_k}=b_k$ ，问是否存在一组解满足所有条件。

若 $k \gt 2$ ，这问题已经被证明是 NP 完全的了，不存在多项式时间内的算法，但若 $k=2$ ，则有一种特殊的算法可以在 $\Omicron(n+m)$ 内解决。



考虑建出一个无向图，每条边都代表这一个「蕴含」限制，即若存在一条边从 $x$ 连向 $y$ ，那么就表示 $x\rarr y$ ，其的真值表如下：

| $x$ | $y$ | $x\rarr y$ |
| --- | --- | ---------- |
| 0   | 0   | 1          |
| 0   | 1   | 1          |
| 1   | 0   | 0          |
| 1   | 1   | 1          |

不难发现，就是要求当 $x$ 为 1 时，$y$ 也要为 1 。

然后就是对这张图填 0 或 1，问是否存在方案，但你可能会说了：全填 0 或者全填 1 不久可以了吗，不要着急，接下来会介绍一些条件，有若干对点必须有且仅有一个点填 1。



我们把一个变量 $x$ 拆开来，变成 $\lnot x$ 和 $x$ ，对于限制 $a \lor b$ ，可以被拆成 $(\lnot a \rarr b )\land (\lnot b \rarr a)$ 。（正确性不难用真值表证明，此处略去）。

建成这张图后，进行缩点，显然在一个强连通分量内的点必须同选 0 或者 1，若 $\lnot x$ 和 $x$ 在同一个强连通分量内，就无解。

否则，一定有解，考虑对缩点后的图进行拓扑排序，若 $\lnot x$ 所在的强连通分量的拓扑序小于 $x$ 所在的强连通分量的拓扑序，则 $x=0$  ，否则 $x=1$ 。



你可能会问，假如出现这样的情况：$a \rarr \lnot a \rarr b \rarr \lnot b$  ，上面的方案不久不合法了吗？

实际上这种情况不会出现，$\lnot a \rarr b$ 原来是 $a \lor b$ ，还有一条边 $\lnot b \rarr a$ ，形成了一个强连通分量，不合法，类似情况同理可以证明不会出现。



实现细节：可以不用跑拓扑序，直接用 Tarjan 跑出来的强连通分量编号，其大的，拓扑序反而小。

[提交记录](https://www.luogu.com.cn/record/109843315)

## [P4151 [WC2011]最大XOR和路径](https://www.luogu.com.cn/problem/P4151)

给定一张无向图，有边权，求一条从 $1$ 到 $n$ 的路径，使得其边权异或和最大。



由于异或操作的良好性质，我们先钦定一条从 $1$ 到 $n$ 的路径，然后考虑往路径上面加环，显然我们从路径上某个点，走到环上的某个点，绕完整个环，再沿着同样的路走回来，只有那个环上的路径异或和会被计入答案，任意一条可能的路径都可以被表示为这样的形式。然后用线性基处理所有环异或起来的最大值即可。正确性可能不显然，感性理解一下。



[提交记录](https://www.luogu.com.cn/record/109624603)

## [P3991 [BJOI2017]喷式水战改](https://www.luogu.com.cn/problem/P3991)

有一个序列，初始为空，有若干个操作，会在序列中第 $p_i$ 个元素个元素后插入 $x_i$ 个第 $i$ 种元素，第 $i$ 种元素有属性 $a_i, b_i, c_i$ ，每次求这个序列的最大价值。

定义这种序列的价值是，你要这个序列分成四段子序列，每个子序列可能为空，第 1 个子序列和第 4 个子序列的价值是所有元素的 $a_i$ 之和，第 2 个子序列是 $b_i$ 之和，第 3 个子序列是 $c_i$ 之和，然后求和。



见到插入操作，考虑用平衡树维护，每个节点维护一段连续的相同种类的元素。

显然，对于每段连续的元素，因为每段可以为空，所以划分到一个子序列中是不劣的。

令 $f_u[i][j]$ 为以 $u$ 为根的节点，整棵子树代表的区间，左分到了第 $i$ （$0\cdots 3$） 个子序列，右分到了第 $j$ 个子序列的最大值，转移方程可以参照代码，每次 `split` 和 `merge` ，左右儿子变更时要更新一下。

另外，插入一个新的段的时候，有可能把原来的一段拆成了两端，具体实现较为繁琐。



[提交记录](https://www.luogu.com.cn/record/109624603)

## [P4577 [FJOI2018] 领导集团问题](https://www.luogu.com.cn/problem/P4577)

给定一棵树，求树上满足如下条件的大小最大的点集：

对于任意点集中的点 $i, j \in S, i \not= j$ ，若在原树中 $u$ 是 $v$ 的祖先，则 $w_u \le w_v$ 。



考虑序列上的LIS的做法，我们也可以叫这道题“树上LIS”。

对于每个节点 $u$ 维护一个序列 $f[u]$  ，其中 $f[u][i]$ 代表着在 $u$ 为根的子树内，所有选取 $i$ 个节点的方案 $T$ 中，$\min_{j\in T} w_j$ 的最大值。

节点 $u$ 的答案就是最大 $i$ ，使得 $f[u][i]$ 合法。

考虑合并所有的 $v \in son_u$  ，显然子树中的答案互相不影响，对于一个 $i$ ，可以贪心地选择出最大 $i$ 个。

考虑加入 $w_u$  ，仿照LIS的做法，从 $f[u]$ 中找出最小的 $i$ 使得 $w_i \ge w_u$ ，显然把 $w_i$ 换成 $w_u$ 不会使得答案更劣，即 $f[u][i] = w_u$ （正确性不太好理解）。



由于 $f[u]$ 有单调型，用 `multiset` 维护 $f[u]$ ，合并 $u$ 的儿子时启发式合并即可，用 `lower_bound` 找到 $w_i \ge w_u$ ，答案就是 $|f[1]|$ 。



[提交记录](https://www.luogu.com.cn/record/109894833)

## [P4591 [TJOI2018]碱基序列](https://www.luogu.com.cn/problem/P4591)

给定一个字符串 $s$ ，和若干组字符串 $t_{ij}$ ，问有多少组方案，从每组字符串中选择一个，首位拼接，能形成了 $s$ 的子串。



用 $f[i][j]$ 表示获得了匹配到 $s_i$  ，用了前 $j$ 组字符串的方案数。
$$
f[i][j] = \sum_{1 \le k \le a_j, l} f[l-1][j-1]\cdot (s[l \cdots i] = t_{jk})
$$
可以用哈希判断 $s[l\cdots i] = t_{jk}$ 。



[提交记录](https://www.luogu.com.cn/record/109896517)

## [P3762 [TJOI2017]龙舟](https://www.luogu.com.cn/problem/P3762)

给定 $b_i$ 和 $a_{ij}$ ，多次询问，给定 $x, M$ ，求 $\frac {b_1\times \cdots \times b_m} {a_{x1}\times \cdots \times a_{xm}} \bmod M$ ，不保证 $M \in \mathbb{P}$ 。

$M, a_{ij}, b_i \lt 2\times 10^{18}, m \le 10000, 询问数 \le 50$



先介绍两个科技算法：



一：$\mathcal{Miller\ Rabin}$ 素数判定。

对于一个很大的数 $n \lt 2\times 10^{18}$ ，判断是否是质数。

先筛掉唯一一个偶素数 $n = 2$ 。

首先会想到费马小定理，若 $n \in P$  ，对于任意的 $a \in [1, n-1]$ ，有：
$$
a^{n-1} = 1 (\bmod n)
$$
但是也有一些合数满足这个等式，我们需要更强的限制。

考虑二次探测，若 $n\in P, x < n$ ：
$$
x^2 = 1 (\bmod p) \rArr x\in \{1,n-1\}
$$
结合费马小定理：
$$
a^{n-1} = (a^{\frac{n-1}2})^2 = 1(\bmod n) \Rarr a^{\frac{n-1}2}\in\{1,n-1\}
$$
若 $a^{\frac{n-1}2}=1$ ，继续考虑 $a^{\frac{n-1}4},a^{\frac{n-1}8},\ldots$ 直到 $\frac {n-1} {2^k}$ 为奇数。

若有一个判断未通过，就肯定不是质数了。

代码：

```cpp
typedef __int128_t lll;
lll Miller_Rabin(lll n)
{
    if (n == 2) return true;
    static const int TEST_TIME = 10;
    uniform_int_distribution<uint64_t> d(2, n - 1);
    for (int _ = 1; _ <= TEST_TIME; _++) {
        lll a = d(eng);
        if (qpow(a, n - 1, n) != 1) return false;
        lll p = n - 1;
        for (; !(p & 1); p /= 2) {
            lll x = qpow(a, p, n);
            if (x * x % n == 1 && x != 1 && x != n - 1) return false;
        }
    }
    return true;
}
```



二：$\mathcal{Pollar\ Rho}$ 算法：

Pollar Rho 算法就是快速找到大整数的一个非 1，非自己的因数算法。

先介绍生日悖论：若有23个人，他们的出生日期均匀分布，那么存在两个人生日相同的概率超过一半（这不难证明 $\frac{365}{365} \times \frac{364}{365} \times \cdots \frac{365-22}{365} \lt 1/2$ ）。

推广一下，若在 $[1, N]$ 之间均匀生成数，那么期望 $\Omicron(\sqrt n)$ 次后会生成重复的数。

考虑 $n$ 的最小因子 $1 \lt p \le \sqrt n$ ，期望随机生成 $\Omicron(n^{1/4})$ 个数后会出现模 $p$ 意义下相同的数，那么这两个数的差的绝对值 $d$ 满足 $\gcd(d, n) > 1$ ，可以直接返回 $\gcd(d, n)$ 。

但假如暴力判断的话，复杂度又回到 $\Omicron(n^{1/4}\ \log n)$ 的了。所以我们设计一种特殊的伪随机数生成器：
$$
\langle 0, f(0), f(f(0))), f^3(0), \ldots\rangle,\ \text{let } f(x) = (x^2+c)\bmod N， c = \operatorname{randint}(1 \cdots n-1)
$$
首先这个生成器会有一个问题，就是可能跳入一个循环出不来了，这时候我们需要换一个 $c$ 。

这里采用 Floyd 判环算法，伪代码如下：
$$
\begin{align*}
1\ \ & x = f(0),\ y = f(f(0))                  \\
2\ \ & \operatorname{while}\ x \not= y         \\
3\ \ & \ \ \ \ \ \ \ \ \text{(Do something.)}  \\
4\ \ & \ \ \ \ \ \ \ \ x = f(x),\ y = f(f(y))  \\
5\ \ & \operatorname{endwhile}
\end{align*}
$$
假如走到了一个环，$y$ 比 $x$ 更快，肯定会在环上某处相遇。

这样有什么好处呢？
$$
|i - j| = 0 (\bmod p) \rArr  |f(i)-f(j)| = |i^2-j^2| = |i-j|\cdot |i+j| = 0 (\bmod p)
$$
说明
$$
\exist i,\ j = f^d(i) \rarr |i-j| = 0 (\bmod p) \\
\Darr \\
\forall i,\ j = f^d(i) \rarr |i-j| = 0 (\bmod p) \\
$$
我们每次可以验证所有「距离」为 $d$ 的点对。

```cpp
lll Pollard_Rho(lll n)
{
    if (Miller_Rabin(n)) return -1;
    if (n == 4) return 2;
    uniform_int_distribution<uint64_t> d(1, n - 1);
    while (true) {
        lll c = d(eng);
        auto f = [c, n](lll y) { return (y * y % n + c) % n; };
        lll x = f(0), y = f(f(0));
        while (x != y) {
            lll z = gcd(abs(x - y), n);
            if (z > 1) return z;
            x = f(x), y = f(f(y));
        }
    }
}
```



为了表示方便，令 $c_i = a_{xi}$

由于一些取模的神秘原因（我不会，但不这么做不对），对于要处理的 $b_i$ 和 $c_i$ ，先消去共同的 $M$ 的质因数。

为了方便，对于 $x = t\cdot y^k$ ，定义 $\operatorname{cnt}(x,y)=k, \operatorname{left}(x,y)=t$ 。

$$
x \in \mathbb{P} \land x|M,\ f_x = \sum_i \operatorname{cnt}(b_i, x) - \sum_i \operatorname{cnt}(a'_i, x) \\

\forall x \in \mathbb{P} \land x|M,\ b_i \larr \operatorname{left}(b_i, x) \\
\forall x \in \mathbb{P} \land x|M,\ c_i \larr \operatorname{left}(c_i, x) \\
$$

若有 $f_x \lt 0$ ，那么无解。

然后答案就是
$$
\frac {\prod_i b_i} {\prod_i c_i} \prod_i x^{f_x}
$$
求逆元需要用欧拉定理算 $\varphi(M)$ 。



[提交记录](https://www.luogu.com.cn/problem/P3762)


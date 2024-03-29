---
title: "网络流中相关习题总结1"
date: 2023-01-25
tags: ["图论", "网络流"]
---

基础概念在 [上一篇](/OI/graph/flow/flow1) 。

个人整理的题单：<https://www.luogu.com.cn/training/273755>

后续内容在 [下一篇](/OI/graph/flow/flow3) 。

记号：

1. 在提及两端点情况下，用 $(c, w)$ 表示一条容量为 $c$ ，费用为 $w$ 的边，下同。

2. 用 $u \rarr v, (c, w)$ 来表示从 $u$ 向 $v$ 连一条容量为 $c$ ，费用为 $w$ 的边，下同， $c$ 有时会省略。

## 类型 1：基本

这个类型建图不需要太多的转换，只需要按照题意建即可。

**重要 trick 拆点**：可以将一个点 $u$ 拆成入点 $u_{in}$ 和出点 $u_{out}$ ，所有连向 $u$ 的 $(v, u)$ 连向 $u_{in}$ ： $(v, u_in)$ ，所有从 $u$ 连的 $(u, w)$ 改为从 $u_{out}$ 连：$(u_{out} , w)$ ，再在 $u_{in}$ 和 $u_{out}$ 之间连边，以此限制路径经过这个点的次数和价值。

**重要 trick 只取一次**：若一个边（点用拆点）上有价值，但只能只取一次，可以这么连边：两个点之间，先连一条 $(1, w)$ ，$w$ 为价值，保证只有一次，再连一条 $(\infty, 0)$ ，让后面可以通过。

### P4013 数字梯形问题

<https://www.luogu.com.cn/problem/P4013>

题意：

一个梯形最上面一行有 $m$ 个数字，每一行比上面一行多 $1$ 个数字，交错排列，从顶部开始，在每个数字可以向左下或者右下走，形成 $m$ 条路径，求以下限制下的路径上所有数字的最大和：

1. 每条路径不能在点或者边相交；
2. 每条路径不能在边上相交；
3. 每条路径可以在点上或者边上相交。

解法：

拆点，源点向最上面的连边 $(\infty, 0)$ ，最下面的向汇点连边 $(\infty, 0)$，中间的点向左下和右下的点连边，情况 1 2 时为 $(1, 0)$ ，情况 3 时为 $(\infty, 0)$，同时每个数字 $i$ 拆成的出点和入点之间，情况 1 连 $(1, i)$ ，否则为 $(\infty, i)$ 。

跑最大费用最大流即可。

### P1042 深海机器人问题

<https://www.luogu.com.cn/problem/P4012>

题意：

有一个网格，和一些机器人，机器人可在格点上向右或向上运动，一些格边上有一些有价值的物品，会被第一个机器人取走，给出若干个起点和终点，每个起点和终点可以让若干个机器人出或入。

要让所有机器人回到终点，求物品价值最大。

解法：

和上一题一样，不过这次由于物品在格边上，故不用拆点，不过注意到一个物品只能被取一次，可以这样操作：一条边从 $u$ 到 $v$ （有方向限制），物品价值 $w$，先建一条 $(1, w)$ ，再建一条边 $(\infty, 0)$ ，可以保证只取一次。然后从 $s$ 向所有的起点连边 $(x, 0)$ ，其中 $x$ 为起点可以让多少个机器人出发，终点同理。

跑最大费用最大流即可。

### P3356 火星探险问题

<https://www.luogu.com.cn/problem/P3356>

题意：

一个网格图，若干个机器人，机器人在格子中，从左上走到右下，有障碍，也有价值为 $1$ 的物品，注意物品只能被取一次。

问在到达终点的机器人个数最大的情况下，被取的物品最大价值是多少，注意没有到终点的机器人取的物品价值不会被计算。

**输出方案：开始时机器人都在左上，每次选择一个机器人和它所做的操作，即输出 `机器人编号 操作(0 为下, 1 为右)` 。**

和上一题几乎一样，用上面介绍的 trick 建图，但这题毒瘤的地方在于要输出方案。

结合网络流的性质可以发现，如果一条边它的边权每减小了 $1$ ，说明有一个机器人走过。考虑到机器人并没有差别，即一个物品不在乎是那个机器人取走的，我们可以一个一个走，从起点开始，寻找一个边权改变了的边（反向边边权 $\gt 0$ ），然后走过它，把边权改回去，根据网络流的性质，不可能存在走着走着走不下去的情况，故这么做是正确的。

输出方案的参考代码：

```cpp
vector<pair<int, int>> edges_new[MAXN];
void export_paths(int lim) // lim 为机器人走到终点的个数，即最大流
{
    // 先建成一个新图，方便处理
    for (int i = 1; i <= n * m * 2; i++) {
        if (i % 2 == 1) continue;
        for (int j = head[i]; j; j = edges[j].nxt) {
            int v = edges[j].v;
            if (edges[j ^ 1].flw && v != i - 1 && v != s && v != t) {
                edges_new[i == s ? s : i / 2].push_back({v == t ? t : (v + 1) / 2, edges[j ^ 1].flw});
            }
        }
    }
    for (int i = 1; i <= lim; i++) {
        int u = 1;
        while (u != n * m) {
            for (auto& p : edges_new[u]) {
                // p.second 为这条边最多经过机器人次数
                if (p.second) {
                    if (p.first == u + 1) {
                        printf("%d %d\n", i, 1);
                    } else {
                        assert(p.first == u + m);
                        printf("%d %d\n", i, 0);
                    }
                    p.second--; // 改边权
                    u = p.first;
                    break;
                }
            }
        }
    }
}
```

### P4066 吃豆豆

<https://www.luogu.com.cn/problem/P4066>

两个 PACMAN 吃豆豆。一开始的时候，PACMAN 在所有豆豆的左下方，PACMAN 走到豆豆处就会吃掉它。PACMAN 只能向右走或者向上走，行走的路线不可以相交。问最多能吃多少个豆豆。

点数 $n \le 2000$ 。

两条路线不能相交的限制并没有什么用处，只要不在有豆豆的地方相交，就可以这样：

![1](./1.svg)

那么就像上一题一样了，可以从任意的点开始和结束（也是为了方便），从 $s$ 向所有的点连边，$t$ 同理。考虑到 $n$ 很大，不能暴力连边，如果有 $u \rarr v, v \rarr w$ 显然就没有必要 $u \rarr w$ 了。

建图详见代码：

```cpp
for (int i = 1; i <= n; i++) scanf("%d%d", &a[i].first, &a[i].second);
sort(a + 1, a + n + 1);

for (int i = 1; i <= n; i++) {
    int minx = INF;
    G.add_edge(ss, i * 2 - 1, 1, 0), G.add_edge(i * 2, tt, 1, 0);
    G.add_edge(i * 2 - 1, i * 2, 1, -1), G.add_edge(i * 2 - 1, i * 2, 1, 0);
    for (int j = i + 1; j <= n; j++) {
        if (a[j].second >= a[i].second && a[j].second < minx) {
            G.add_edge(i * 2, j * 2 - 1, INF, 0);
            minx = a[j].second;
        }
    }
}
```

## 类型 2：进阶——不明显的图

有些网络流题目的图不会那么明显。

### P2770 航空路线问题

给定若干个城市，从西到东排列，城市之间有一些航线，判断是否存在一条最长航线满足以下要求，若满足，输出最多访问的城市个数和方案：

1. 从最西端的城市出发，从西向东到最东端的城市，在从东向西回到最西端的城市。

2. 除了最东端和最西端的城市，每个城市只能访问一次。

显然可以建图，把城市之间的航线看作边，要求的航线看作路径。

一条折回来的航线不好考虑，改为两条从起点出发、不相交的自西向东的航线。

每个点只访问一次的限制可以拆点解决，最西边、最东边的点可以访问两次，其他的点只能访问一次。

故综上，建图方案为：

$s \rarr 1_{in}, (2, 0)$ ， $1_{in} \rarr 1_{out}, (2, 1)$ ； $n_{in} \rarr t, (2, 0)$ ， $n_{in} \rarr 1_{out}, (2, 1)$ ；

$2 \le i \le n-1, i_{in} \rarr i_{out}, (1, 1)$

航线 $(x, y)$ 连边 $x_{out} \rarr y_{in}, (1, 0)$

求最大费用最大流。

注意：只有两个点时，最大流为 $1$ ，但有可行方案（ $1 \rarr 2 \rarr 1$ ），注意特判。

输出方案在之前有提及，就是跳被修改过的边就可以了。

### P2766 最长不下降子序列问题

<https://www.luogu.com.cn/problem/P2766>

给定一个序列，

1. 计算其最长不下降子序列的长度 $s$ 。

2. 如果每个元素只允许使用一次，计算从给定的序列中最多可取出多少个长度为 $s$ 的不下降子序列。

3. 如果允许在取出的序列中多次使用 $x_1$ 和 $x_n$ ，计算序列中最多能取出多少个不同的长度为 $s$ 的不下降子序列。

对于一个序列 $a = a_1, a_2, \cdots a_n$ ，如果有一个序列 $p$ 满足 $1 \le p_1 \lt p_2 \lt \cdots \lt p_k \le n$ ，那么序列 $b = a_{p_1}, a_{p_2}, \cdots a_{p_k}$ 为序列 $a$ 的子序列。

对于序列 $a$ 的子序列 $b$ 和 $c$ ，分别是由 $p$ 和 $q$ 构造的，称 $b$ 和 $c$ 不同，当且仅当 $p$ 不等于 $q$ ，即 $|p| \not= |q| 或 \exists 1\le i \le |p|, p_i \not= q_i$

求最长不下降子序列的长度可以由 dp 完成，设 $f_i$ 为最后一个数是 $a_i$ 的最长不降子序列长度，那么 $f_i = \max_{j=1}^{i-1} f_j + 1, a_i \ge a_j$ ， $s = \max f_i$

**我们把选出一条长度为 $s$ 的不降子序列看作选出一条路径。**

把一个点 $i$ 拆点，从入点向出点连一条边，容量为 $1$ ，保证只有一条路径，即一个序列选择这个点。

然后从源点向所有 $f_i = 1$ 的 $i$ 的入点连边，从所有 $f_i = s$ 的 $i$ 的出点向汇点连边，然后对于所有 $i$ 的出点向 $f_j = f_i + 1, j \gt i$ 的入点连边。以上所有的边权都是 $1$ 。

![以样例为例](./2.png)

跑一次最大流，就是第二问的答案。

对于第三问，只要把从 $s \rarr 1_{in}, 1_{in} \rarr 1_{out}, n_{in} \rarr n_{out}, n_{out} \rarr t$ 的边的容量改为 $\infty$ 即可。

第三问中的「不同」的限制只是限制位置不同，而这已经被 $i \rarr j$ 之间的容量为 $1$ 的条件限制住了，故不用额外考虑。

### P3358 最长k可重区间集问题

<https://www.luogu.com.cn/problem/P3358>

有若干个开区间，给定他们的左右端点，要求从其中选出一些，使得对于任意的 $x_0$ ，至多 $k$ 个开区间包含 $x_0$ 。

求最长的区间长度之和。

考虑要如何处理这两个限制：最多 $k$ 个区间，和如果选了这一个区间，就会影响从左到右的若干个点。

考虑这么建图：

对于区间 $(l, r)$ ，从 $l$ 向 $r$ 连一条容量为 $1$ ，费用为 $|(l, r)|$ 的边，在相邻的点从左向右连一条容量为 $k$ ，费用为 $0$ 的边。然后从源点向最左边的点连一条容量为 $k$ ，费用为 $0$ 的边，从最右边的点向汇点连一条容量为 $k$ ，费用为 $0$ 的边，然后跑最大费用最大流。

![以样例为例](./3.svg)

为什么是对的呢？考虑将选择的区间分成 $k$ 组，每组之间两两不交，就相当于从源点向汇点找 $k$ 条路径，每条路径不同时经过两个区间，每个区间只用一次，恰好，我们的建图就能满足这个条件，同时显然满足条件的情况下，多加入一个区间会使答案更优，所以最大费用最大流是正确的。

### P3357 最长k可重线段集问题

<https://www.luogu.com.cn/problem/P3357>

有若干的开线段，给定两端点的坐标，要求从其中选出一些，使得对于任意的 $x_0$ ，至多 $k$ 个线段和 $x = x_0$ 相交。

求最长的线段长度（一条线段的距离定义为两端点的欧几里得距离下取整）。

显然和上一题很像，对于每一条线段，显然可以将他们看作在 $x$ 轴上的投影，但注意有特殊情况，即垂直于 $x$ 轴的线段，它们按照上一题的建图方式建出来是不能限制只取 $k$ 个的。

考虑拆点：

假如一条线段的两端点 $x$ 坐标不同，从左端点的 $x$ 坐标对应的出点连到右端点的 $x$ 坐标对应的入点。如果相同，就从 $x$ 坐标对应的入点连到出点，然后把剩下的串起来就可以了：$i_{in} \rarr i_{out} \rarr (i+1)_{in} \rarr (i+1)_{out} \rarr \cdots$ 。

### P3980 志愿者招募

有一个工作共 $n$ 天，第 $i$ 天的工作需要 $a_i$ 人，同时可以有 $m$ 种志愿者，第 $i$ 种志愿者可以从 $s_i$ 天工作到 $t_i$ 天（每天都来，不能拆开来），每人共需要 $c_i$ 的工钱。

问最小花费。

这题和上两题类似，不过上两题是至多，这题至少。

考虑用流量限制人数，对于每个 $i$ ，从 $s_i$ 向 $t_i + 1$ 连一条 $(\infty, c_i)$ 的边，同时取一个较大的数 $MAX$  ，从 $i$ 向 $i + 1$ 连一条 $(MAX - a_i, 0)$ 的边，然后连 $s \rarr 1, (MAX, 0)$ ， $n + 1 \rarr t, (MAX, 0)$ ，然后跑最小费用最大流。

为了让网络跑到最大流 $MAX$ ，对于 $i \rarr i + 1$ ，其容量为 $MAX - a_i$ ，就要在经过这段的志愿这的边至少流过 $a_i$ ，这就满足了每天要有 $a_i$ 个人的限制了。

![解释](./4.svg)

## 类型 3：分层图

把分层图的思想引入网络流，可以记录更多的信息。

### P4099 汽车加油行驶问题

<https://www.luogu.com.cn/problem/P4099>

有一个网格，一辆汽车在格点和边上行驶。汽车从起点 $(1, 1)$ 出发驶向右下角终点 $(n, n)$。

在若干个格点处，设置了油库，可供汽车在行驶途中加油。汽车在行驶过程中应遵守如下规则:

1. 出发时汽车已装满油，装满油的汽车能够行驶 $k$ 条网格边。

2. 汽车若向左或向上行驶，需付费用 $B$ 。

3. 汽车在行驶过程中遇油库则应加满油并付加油费用 $A$ 。

4. 在需要时可在网格点处增设油库，并付增设油库费用 $C$ （不含加油费用$A$ ）。

问从起点行驶到终点的最小费用。

$n \le 100, k \le 10$

不能用边的容量等条件限制汽车的油亮，考虑使用分层图。

连边如下，点 $(x, y, p)$ 汽车在 $(x, y)$ ，还有 $p$ 的油。

起点和终点：$s \rarr (1, 1, k), (1, 0);\ (n, n, p) \rarr t, (1, 0)$

中间的点：$1 \le x \le n, 1 \le y \le n$ ， $(x', y')$ 与 $(x, y)$ 相邻，需要费用 $w$ （ $w$ 为 $B$ 或 $0$ ）。

如果 $(x, y)$ 有油库，则 $(x, y, p) \rarr  (x', y', k - 1), (\infin, w + A)$

否则，若不设油库， $(x, y, p) \rarr  (x', y', p - 1), (\infin, w)$ ，若设油库 $(x, y, p) \rarr  (x', y', p - 1), (\infty, w + C)$ （绕一圈回到原来的地方显然是不优的，所以不必担心设油库的费用 $C$ 被算两次）。

### P2754 家园 / 星际转移问题

<https://www.luogu.com.cn/problem/P2754>

把若干个人从地球转移到月球，有 $n$ 个太空站， $m$ 艘太空船在地球，月球，太空站之间运送旅客，第 $i$ 个太空站编号 $i$ ，为方便，地球编号 $0$ ，月球编号 $-1$ 。

设第 $i$ 艘太空船，若它的路线为 $(x_1, x_2, \cdots x_n)$ ，那么它会依次停靠 $x_1, x_2, \cdots, x_n, x_1, x_2, \cdots, x_n, \cdots$ 号太空站（或地球或月球），它可以容纳 $h_i$ 个人。

问至少多久可以将所有人从地球运到月球。

使用分层图的思路，设 $(i, k)$ 为编号为 $i$ 的地方在第 $k$ 时刻的节点。

从第 $0$ 个时刻开始，每个时刻 $k$ ，对于每个地方，新建一个节点，$(i, k-1) \rarr (i, k), (0)$ （月球除外， $(-1, k) \rarr (-1, k-1)$ ），然后对于每个飞船 $i$ ，设其上个时刻所在的位置为 $p_{i, k-1}$ ，这个位置在 $p_{i, k}$ ，那么 $(p_{i, k-1}, k-1) \rarr (p_{i, k}, k), (h_i)$ 。

不断重复这个操作，知道这个图的最大流大于等于人数即可（添加新的边，最大流肯定是不降的，所以不用清空，直接跑就行了）。

### P2053 修车

<https://www.luogu.com.cn/problem/P2053>

有 $n$ 辆车，分给 $m$ 个人修，第 $i$ 个人修第 $j$ 辆车需要 $t_{i,j}$ 的时间，第 $i$ 辆车的等待时间是修它的时间加上选择修它那个人修之前的车的等待时间之和，求最小等待时间。

形式化地说：

第 $i$ 个人修 $k_i$ 辆车，分别为 $a_{i, 1}, a_{i, 2}, \cdots a_{i, k_i}$ ，其中 $\sum k_i = n$ ，$a_{1, 1}, a_{1, 2} \cdots a_{1, k_1}, \cdots a_{i, j} \cdots a_{m, k_m}$ 互不相同，且在 $1$ 到 $n$ 之间。

求

$$
\min \frac {\sum_{i=1}^m \sum_{j=1}^{k_i} \sum_{l=1}^j t_{i, a_{i, l}}} n
$$

肯定先求总时间最少。

考虑使用贡献，和从后往前安排顺序，如果第 $i$ 辆车，找 $j$ 修，从后往前第 $k$ 个，那么它对答案的贡献就是 $t_{j, i} k$ 。

![解释](./5.svg)

二分图，左边是 $n$ 个人，右边是 $m * n$ 个点，右边的点 $(j, k)$ 表示找 $j$ ，从后往前第 $k$ 个修，然后左边的 $i$ 向右边的 $j$ 连边 $(1, t_{j, i}k)$ ，然后从源点向左边连 $(1, 0)$ ，从右边向汇点连 $(1, 0)$ 。

跑最小费用最大流就可以了。

### P2050 美食节

这一题和上一题类似，不过数据范围较大。

我们发现不是所有的 $(j, k)$ 都是有用的，所以我们最开始只建 $(j, 1)$ ，然后假如 $(j, k)$ 被用了，再建 $(j, k + 1)$ 。

可以参考代码：

```cpp
inline pair<int, int> dinic()
{
    int fsum = 0, csum = 0;
    while (spfa()) {
        while (true) {
            pair<int, int> p = dfs(S, INF);
            int x = p.first, y = p.second;
            fsum += x, csum += y;
            if (!x) break;
        }
        for (register int j = 1; j <= m; j++) {
            if (vis1[n + id(j, top[j])] && top[j] < sum) {
                // (j, k) 被用了
                top[j]++;
                add_edge(n + id(j, top[j]), T, 1, 0);
                for (register int i = 1; i <= n; i++)   
                    add_edge(i, n + id(j, top[j]), 1, a[i][j] * top[j]);
                // 连接 i
            }
        }
    }
    return make_pair(fsum, csum);
}
```

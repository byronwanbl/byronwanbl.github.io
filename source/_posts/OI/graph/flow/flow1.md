---
title: "网络流中的基本概念"
date: 2023-01-24
tags: ["图论", "网络流"]
---

介绍一些网络流中的基本概念，实现方法。

对于题目的总结在[下一篇](/OI/graph/flow/flow2) 。

## 什么是网络流

有一个有向图，其中有两个特殊的点，源点和汇点，你可以通过图中的点和边，从源点向汇点流流量，边有流量限制，然后解决这个图在一定情况下的流量问题（稍后会提及）。

你也可以把它理解为一个供水网络，源点就是水厂，汇点就是你家，每个点就是中转站，有管道连接一些点，管道有粗细，能流过的最大流量不同。

形式化来说：

在一个有向图 $G = (V, E)$ 中，每条边 $(u, v) \in E$ 都有一个权值 $c(u, v)$ ，表示这条边的容量，同时为了方便，通常认为，对于 $(u, v) \not\in E$ ，$c(u, v) = 0$ 。

同时，每条边 $(u, v)$ 还有个值 $f(u, v)$ ，其为这条边上的流量，它需要满足：

1. $f(u, v) \le c(u, v)$ 不得超出最大流量限制；
2. 对于所有的 $u \not= s, t$ 满足 $\sum_{(v, u)\in E} f(v, u) = \sum_{(u, w)} f(u, w)$ 输入和输出的流量相等，流量守恒；
3. $\sum_{(s, u) \in E} f(s, u) = \sum_{(v, t) \in E} f(v, t)$ 从源点流出的流量等于汇点流入的流量；

接下来，就是网络流中一个非常精妙的地方，对于一条边 $(u, v)$ ，我们建一条反向边 $(v, u)$ ，满足 $f(v, u) = -f(u, v)$ ，即一条边的反向边的流量是他的相反数。

这有什么用呢？它给了我们一个反悔的机会，可以让我们撤销一条边的选择，这在最大流的实现中会提及。

显然加上这个条件后，性质 2. 仍满足。

## 最大流

对于一个网络流，让它的流量，即 $\sum_{(s, u) \in E} f(s, u)$ 最大，这就是最大流。

第一反应是可以用一个贪心的做法，不断找到一条从源点到汇点的可行路径（流量比容量小），然后修改路径上的边。

即对于路径 $P = \{(s, u_1), (u_1, u_2), \cdots, (u_p, t)\}$ ，设其的流量 $f_P = min_{(u, v) \in P} f(u, v)$ ，然后 $(u, v) \in P, f(u, v) \larr f(u, v) + f_P$ 。

但这样有个问题，例如对于下面的图

![1](./1.svg)

显然最大流应该是 $4$ ，但假如找到了这样一条路径：

![2](./2.svg)

就没有办法继续了，此时的最大流 $2$ 显然是错误的。

现在就要反向边出场了：（只展示了中间的边的反向边）

![3](./3.svg)

然后又可以有新的路径了：

![4](./4.svg)

现在最大流就是 $4$ 。

可以看出反向边的作用就是提供一个撤销错误的贪心决定的机会，以实现全局最优解。

用 bfs 寻找增广路，然后进行增广，就是 Edmonds-Karp 算法，复杂度 $O(nm^2)$ ，其中 $n$ 为点数， $m$ 为边数，下同。

### Dinic 算法

但一条一条路径（可以称为增广路）找实在太慢了，可不可以一次找多条增广路呢？

这就是 Dinic 算法，先用 bfs 分层，然后用 dfs ，每次只能转移到下一层，向下递归时，寻找增广路，回来时，完成对增广路上边容量的修改，不断重复直到不联通（不存在一条合法路径）。

代码参考：

模板题：[「洛谷」P3376](https://www.luogu.com.cn/problem/P3376)

```cpp
struct graph_t {
    struct edge_t { // 链式前向星
        int v, nxt, flw;
        // 实现上为了方便，直接修改流量
    } edges[MAXM];
    int head[MAXN], e_cnt = 1;
    // u^1 得到 u 的反向边

    void _add(int u, int v, int f) { edges[++e_cnt] = {v, head[u], f}, head[u] = e_cnt; }
    void add(int u, int v, int f) { _add(u, v, f), _add(v, u, 0); }

    int dis[MAXN], now[MAXN]; // 当前弧优化，从未流尽的边开始考虑
    bool vis[MAXN];
    bool bfs()
    {
        memset(dis, 0x3f, sizeof(dis));
        memcpy(now, head, sizeof(now));
        queue<int> q;
        q.push(s), dis[s] = 0;
        while (q.size()) {
            int u = q.front();
            q.pop();
            for (int i = head[u]; i; i = edges[i].nxt) {
                if (edges[i].flw && dis[edges[i].v] > dis[u] + 1) {
                    // 分层
                    dis[edges[i].v] = dis[u] + 1;
                    q.push(edges[i].v);
                }
            }
        }
        return dis[t] < INF;
    }

    int dfs(int u, int flow)
    {
        if (u == t || !flow) return flow;
        vis[u] = true;
        int maxflow = 0;
        for (int i = now[u]; i && flow; now[u] = i = edges[i].nxt) {
            int v = edges[i].v;
            if (!vis[v] && edges[i].flw && dis[v] == dis[u] + 1) {
                // 一层一层来
                int k = dfs(v, min(flow, edges[i].flw));
                if (!k) dis[v] = INF;
                // 删去流尽的边
                maxflow += k, flow -= k, edges[i].flw -= k, edges[i ^ 1].flw += k;
                // 应用修改
            }
        }
        vis[u] = false;
        return maxflow;
    }

    int dinic()
    {
        int maxflow = 0, x = 0;
        while (bfs()) {
            while ((x = dfs(s, INF))) maxflow += x;
        }
        return maxflow;
    }
} G;
```

#### 时间复杂度证明

最坏时间复杂度 $O(n^2m)$ ，大多数情况远远跑不满 。

考虑每次增广，在使用当前弧优化的情况下，对于每个点，显然当前弧 `now` 至多变化 $m$ 次，共 $n$ 个点，故单次时间复杂度 $O(nm)$ 。

对于汇点 $t$ ，其的层数 `dis[t]` 在一次增广后一定变大至少 $1$ （假如未变大，说明有一条应被考虑的增广路未被考虑），显然层数最大 $n - 1$ ，故 Dinic 算法的时间复杂度 $O(n^2m)$ 。

### HLPP 算法（不重点介绍）

Highest-Label-Preflow-Push 最高标号预流推进算法，可以在 $O(n^2\sqrt m)$ 时间复杂度计算出最大流。

首先来看一下预流推进算法的思想。

这个算法的不同之处就是每个点可以存一些流量（即输入流量减去输出流量），称作超额流，同时每个节点都有高度，可以把存储的超额流推送到有边连接的，高度比他小点（push 操作），但假如没有这样的点，就要更改这个节点的高度，让它的超额流可以流出去（relabel 重贴标签操作），最后所有除了源、汇点之外的点都没有超额流了（每个点无法流到汇点的超额流会被退回到源点），此时进入汇点的流量就是这个网络的最大流。

例子（来自 oi-wiki）：

![来自 oi-wiki](https://oi-wiki.org/graph/flow/images/1152.png)

具体实现结合代码注释理解：

模板题 [「洛谷」P4722](https://www.luogu.com.cn/problem/P4722)

```cpp
int n, m, s, t;

struct graph_t {
    struct edge_t {
        int v, nxt, flw;
    } edges[MAXM];
    int head[MAXN], e_cnt = 1;

    void _add_edge(int u, int v, int f) { edges[++e_cnt] = {v, head[u], f}, head[u] = e_cnt; }
    void add_edge(int u, int v, int f) { _add_edge(u, v, f), _add_edge(v, u, 0); }

    int h[MAXN], gap[MAXN + 100], val[MAXN];
    bool in_q[MAXN];
    priority_queue<pair<int, int>> q;

    // 从汇点开始标记高度
    bool bfs()
    {
        memset(h, 0, sizeof(h));
        queue<int> q;
        q.push(t);
        h[t] = 0;
        while (q.size()) {
            int u = q.front();
            q.pop();
            for (int i = head[u]; i; i = edges[i].nxt) {
                int v = edges[i].v;
                if (edges[i ^ 1].flw && h[v] > h[u] + 1) {
                    h[v] = h[u] + 1;
                    q.push(v);
                }
            }
        }
        if (h[s] < 1e9) {
            for (int i = 1; i <= n; i++) {
                if (h[i] >= 1e9) h[i] = n + 100;
            }
            return h[s] = n, true;
        } else return false;
    }

    // 向高度正好小 1 的点流超额流
    void push(int u)
    {
        for (int i = head[u]; i && val[u]; i = edges[i].nxt) {
            int v = edges[i].v;
            if (edges[i].flw && h[v] == h[u] - 1) {
                int k = min(edges[i].flw, val[u]);
                edges[i].flw -= k, edges[i ^ 1].flw += k;
                val[u] -= k, val[v] += k;
                if (v != s && v != t && !in_q[v]) q.push({h[v], v}), in_q[v] = true;
            }
        }
    }

    // 重贴标签，让 h 刚好比相连的 h 最小的大 1
    void relabel(int u)
    {
        h[u] = 1e9;
        for (int i = head[u]; i; i = edges[i].nxt) {
            int v = edges[i].v;
            if (edges[i].flw && h[u] > h[v] + 1) h[u] = h[v] + 1;
        }
        if (h[u] >= 1e9) h[u] = n + 100;
    }

    int hlpp()
    {
        if (!bfs()) return 0;
        // 高度为 h 的有多少个
        for (int i = 1; i <= n; i++) gap[h[i]]++;
        // 从源点推出，忽略高度
        for (int i = head[s]; i; i = edges[i].nxt) {
            int v = edges[i].v;
            if (edges[i].flw) {
                int k = edges[i].flw;
                edges[i].flw -= k, edges[i ^ 1].flw += k;
                val[v] += k;
                if (v != s && v != t && !in_q[v]) q.push({h[v], v}), in_q[v] = true;
            }
        }

        while (q.size()) {
            int u = q.top().second;
            // 取出高度最高的点
            q.pop();
            in_q[u] = false;
            push(u);
            if (val[u]) { // 没流完
                gap[h[u]]--;
                if (!gap[h[u]]) {
                    // gap 优化
                    // 比它高的点不能把流量推送到 t 了
                    // 高度设为 n + 1 尽快推送回 s
                    for (int i = 1; i <= n; i++) {
                        if (i != s && i != t && h[i] > h[u] && h[i] < n + 1) h[i] = n + 1;
                    }
                }
                relabel(u);
                // 重贴标签
                gap[h[u]]++;
                q.push({h[u], u});
                in_q[u] = true;
            }
        }
        return val[t];
    }
} G;
```

## 最小割

一个有向图，有边权，它的最小割就是切断一些边，把这个图切成两半，让源点 $s$ 和汇点 $t$ 不存在一条路径，同时割断的边边权和最小。

形式化来说：

对于图 $G = (V,E)$ ，其的割为将点划分为两部分，$S$ 和 $T = V - S$ ，有 $s \in S$, $t \in T$ 。

一个割 $(S, T)$ 的容量为 $c(S, T) = \sum_{u \in S, v \in T} c(u, v)$ 。

图 $G$ 的最小割就是求一个割，使其容量最小。

一个图的最小割等于其的最大流，感性理解，不作证明。

## 最小费用最大流

一个图 $G = (V, E)$ ，每条边上还有一个单位流量的费用 $w(u, v)$ ，在满足流量最大的情况下，同时使费用最小。

即在满足 $\sum_{(s, u) \in E} f(s, u)$ 的情况下，让 $\sum_{(u, v) \in E} f(u, v) w(u, v)$ 最小（只考虑原图的边，不考虑反向边）。

对于边 $(u, v)$ 的反向边 $(v, u)$ ，令 $w(v, u) = -w(u, v)$ ，使得流经反向边时能够撤销贡献。

只要把 Dinic 算法的 bfs 改成 spfa ，每次寻找最短的路径（ $w$ 最小）增广。

参考代码：

模板题 [「洛谷」P3381](https://www.luogu.com.cn/problem/P3381)

``` cpp
struct graph_t {
    struct edge_t {
        int v, nxt, flw, cst;
    } edges[MAXM];
    int head[MAXN], e_cnt = 1;
    void _add_edge(int u, int v, int f, int c) { edges[++e_cnt] = {v, head[u], f, c}, head[u] = e_cnt; }
    void add_edge(int u, int v, int f, int c) { _add_edge(u, v, f, c), _add_edge(v, u, 0, -c); }

    int dis[MAXN], now[MAXN];
    bool vis[MAXN];
    bool spfa() // 最短路
    {
        memset(dis, 0x3f, sizeof(dis)), memcpy(now, head, sizeof(now));
        queue<int> q;
        q.push(s), dis[s] = 0;
        while (q.size()) {
            int u = q.front();
            q.pop();
            vis[u] = false;
            for (int i = head[u]; i; i = edges[i].nxt) {
                int v = edges[i].v;
                if (edges[i].flw && dis[v] > dis[u] + edges[i].cst) {
                    dis[v] = dis[u] + edges[i].cst;
                    if (!vis[v]) vis[v] = true, q.push(v);
                }
            }
        }
        return dis[t] < INF;
    }

    pair<int, int> dfs(int u, int flow) // 和最大流差不多
    {
        if (u == t || !flow) return {flow, 0};
        int maxflow = 0, mincost = 0;
        vis[u] = true;
        for (int i = now[u]; i && flow; now[u] = i = edges[i].nxt) {
            int v = edges[i].v;
            if (edges[i].flw && dis[v] == dis[u] + edges[i].cst && !vis[v]) {
                auto p = dfs(v, min(flow, edges[i].flw));
                int x = p.first, y = p.second;
                if (!x) dis[v] = INF;
                flow -= x, maxflow += x, edges[i].flw -= x, edges[i ^ 1].flw += x;
                mincost += y + x * edges[i].cst; // 更新贡献
            }
        }
        vis[u] = false;
        return {maxflow, mincost};
    }

    pair<int, int> dinic()
    {
        int maxflow = 0, mincost = 0;
        while (spfa()) {
            while (true) {
                auto p = dfs(s, INF);
                maxflow += p.first, mincost += p.second;
                if (!p.first) break;
            }
        }
        return {maxflow, mincost};
    }
} G;
```

最大费用最大流只要把最短路改为最长路即可。

## 上下界网络流

上下界网络流就是在普通的网络流上，再对每条边加上一个限制 $l(u, v)$ ，流量需满足 $l(u, v)\le f(u, v) \le c(u, v)$ 。

### 无源汇上下界可行流

没有源点汇点，即所有的点都满足输入流量等于输出流量，且每条边的流量都满足上下界限制，问是否存在这样的方案。

考虑每个边已经流满了下界，同时下文中的源点和汇点，指在新的图中新建的超级源点 $s'$ 和超级汇点 $t'$ 。

我们构造一个只有流量上限的新图 $G$ ，对于原图中每条边 $(u, v)$ ，在新图中连接 $(u, v)$ ，同时其的流量限制为 $f'(u, v) = f(u, v) - l(u, v)$ 。

接下来考虑点 $u$ ，设在图 $G$ 中连向 $u$ 的所有边的下界和 $in(u) = \sum_{(v, u) \in G} l(u, v)$, $out(u)$ 为 $u$ 连向其他点的下界和 $out(u) = \sum_{(u, w)\in G} l(u, w)$ 。

若 $in(u) - out(u) > 0$ ，则从源点向 $u$ 连容量为 $in(u) - out(u)$ 的边。

若 $in(u) - out(u) < 0$ ，则从 $u$ 向汇点连权值为 $out(u) - in(u)$ 的边。

然后跑最大流即可，所有连向源点和汇点的边跑满，则可行。

为什么是正确的呢？

图 $G'$ 满足流量守恒，我们要让图 $G$ 也满足流量守恒。

设源点向 $u$ 连边的边权为 $a(u)$ ，$u$ 向汇点连边的边权为 $b(u)$ 。

$$
\begin{aligned}
\sum_{v \not= t'} f'(v, u) + a(u) &= \sum_{w \not= t'} f'(u, w) + b(u) \\
\sum f(v, u) - \sum l(v, u) + a(u) &= \sum f(u, w) - \sum l(u, w) + b(u) \\
a(u) - b(u) + \sum l(u, w) - \sum l(v, u) &= \sum f(u, w) - \sum f(v, u) = 0 \\
a(u) - b(u) &= \sum l(v, u) - \sum l(u, w) = in(u) - out(u) \\
\end{aligned}
$$

若 $in(u) - out(u) > 0$ ，$b(u) = 0, a(u) = in(u) - out(u)$

若 $in(u) - out(u) < 0$ 同理。

### 有源汇上下界可行流

从原图源点向汇点连一条，下界 $0$ ，上界 $\infty$ 的边，转化为无源汇上下界可行流。

此时可行流为新图中原来的汇点和原来的源点之间连边的流量。

证明：

考虑源点 $s$ ，其连向超级汇点的边 $b(s) = out(s) = \sum l(s, u)$

$$
\begin{aligned}
\sum_{u\not=t} f'(s, u) + b(s) &= f'(v, u) \\
\sum f(s, u) - \sum l(s, u) + \sum l(s, u) &= f'(v, u) \\
\sum f(s, u) &= f'(v, u) \\
\end{aligned}
$$

### 有源汇上下界最大流

先对图 $G$ 跑一边可行流（得到新图 $G'$ ），然后删去所有的 $(s', u)$ 和 $(u, t')$ 还有 $(t, s)$ ，以 $s$ 为源点， $t$ 为汇点，跑一遍最大流，加起来，即为答案。

跑可行流时，就满足了流量限制和流量守恒，再跑最大流，不破坏流量限制，同时也能保证流量守恒，和流量最大。

### 有源汇上下界最小费用流

注意：不一定满足最大流。

对原图 $G$ 按照有源汇上下界可行流的方式建出图 $G'$ ，其中图 $G'$ 中的 $w'(u, v) = w(u, v)$ ，即和原来的图的费用一样，$t$ 连向 $s$ ，连向 $t'$ 和 从 $s'$ 连出的其他点费用为 $0$ ，然后对这个图跑最小费用最大流，在加上图 $G$ 中的每条边下界乘上边权即可。

例题：[「洛谷」P4043](https://www.luogu.com.cn/problem/P4043)

## 扩展阅读 & 参考资料

- [「oi-wiki」](https://oi-wiki.org/graph/flow/)

- 算法竞赛进阶指南（李煜东著）0x6A 网络流初步

- 算法导论（机械工业出版社，*Thomas H. Cormen*, *Charles E. Leiserson* 等著），第六部分图算法，第 26 章最大流

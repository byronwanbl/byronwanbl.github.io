---
title: "Cayley's Formula 凯莱公式"
date: 2023-01-11
tags: ["图论", "Prufer 序列", "凯莱公式"]
---

对于一棵 $n$ 个点的完全图，它的生成树共 $n^{n-2}$ 种。

## 证明 I ： Prüfer 序列

为了方便，下文写作 Prufer 序列。

Prufer 序列可以将一棵节点有编号的树与整数序列一一对应的方法。

Prufer 序列多用于组合计数问题，比如要本文证明的 Cayley's Formula 。

### 从树构造 Prufer 序列

对于一棵树，每次选取编号最小的且只有一条边相连的节点，删除它，并把它相连的点加入到序列中，共重复 $n-2$ 次。

#### 实现

显然可以用堆维护最小的点，时间复杂度 $O(n\log n)$

考虑每删除一个点，只有它的父亲（以 $n$ 为根）可能成为新的叶子节点。

用一个指针 $p$ 维护最小的叶子节点。

每删除一个点，看看它的父亲是否是叶子节点，并且比 $p$ 要小，如是，删除它的父亲，并重复这个操作。

然后 $p$ 自增，直到找到下一个叶子节点。

代码如下：

```cpp
void tree2prufer()
{
    for (int i = 1; i < n; i++) deg[fa[i]]++; // 为了方便，度数 -1

    for (int i = 1, j = 1; i <= n - 2; i++, j++) {
        while (deg[j]) j++; // 找到下一个叶子节点
        pru[i] = fa[j];
        for (; i <= n - 2 
                 && !(--deg[pru[i]]) // 父亲是否是叶子
                 && pru[i] < j; // 父亲是否比 p 小
            i++) {
            pru[i + 1] = fa[pru[i]];
        }
    }
}
```

### Prufer 序列的性质

对于度数为 $d$ 的节点，其在 Prufer 序列中出现 $d-1$ 次。

证明：

对于最后只剩两个点，显然他们各自度数为 $1$ ，满足。

否则考虑按照删点的相反顺序加点，每假如一个点 $u$ ，设相连的点为 $v$ ，那么 $v$ 在 Prufer 序列中出现次数多 $1$ 次，同时 $v$ 的度数加 $1$ ，$u$ 的度数为 $1$ 。

对于度数为 $d$ 的点，经历 $d-1$ 次如上加点，故在 Prufer 序列中出现 $d-1$ 次。

### 从 Prufer 序列构造树

考虑以构造 Prufer 序列相同的方式构造树。

根据 Prufer 序列的性质，我们可以算出每个节点的度数。

然后每次找最小的度数为 $1$ 的点，把他和序列头的点连起来，然后对它和序列头的节点度数减 $1$ ，然后删去序列头。
最后把剩下的两个点连起来。

#### 实现

和构造 Prufer 序列操作相同。

在最开始时可以往 Prufer 序列后面加 $n$ 以方便实现。

```cpp
void prufer2tree()
{
    for (int i = 1; i <= n - 2; i++) deg[pru[i]]++;
    pru[n - 1] = n;
    for (int i = 1, j = 1; i <= n - 1; i++, j++) {
        while (deg[j]) j++;
        fa[j] = pru[i];
        for (; i <= n - 1 && !(--deg[pru[i]]) && pru[i] < j; i++) fa[pru[i]] = pru[i + 1];
    }
}
```

### 最后

由上，感性理解 Prufer 序列是 $n$ 个点的无根树和长为 $n-2$ ，每个元素为 $1$ 到 $n$ 之间的序列的双射。

所以 $n$ 个点的完全图的生成树映射到 Prufer 序列，方案数就是 $n^{n-2}$ 。

## 证明 II：对一个计数问题用两种方法

思路大概可以表示为

```plain
Cayley's Formula ---> 方法 A ---> 问题 Q <--- 方法 B
        ^                          |
        |                          |
        +--------------------------+
```

问题是，有多少个有向边序列可以构成一棵有根树。
（有向边序列是： $\{(x_i, y_i)\}_{i = 1}^n$，$x_i$ 为父亲，$x_i \not= y_i$ ）

### 方法 A：使用 Cayley's Formula

``` plain
选定一棵生成树 ---> 选定一个为根 ---> 排列边
```

设选定一个生成树的方案数为 $S_n$ ，答案为 $A_n$ 。
选根共 $n$ 种方案，排列 $n-1$ 条边 $(n-1)!$ 种方案。

$$
A_n = S_n \cdot n \cdot (n-1)! = S_n \cdot n!
$$

### 方法 B：一个一个加

假设已经有了 $k$ 个森林，当前考虑 $(u, v)$ 。

$u$ 可以从 $n$ 个中选取一个， $v$ 则从除去 $u$ 的 $k-1$ 棵树的根中选取一个。

故方案数为 $n(k-1)$ 。

最开始有 $n$ 棵树，每加一条边少一棵。

故答案为：

$$
\begin{aligned}
A &= n(n-1) \cdot n(n-2) \cdot \ldots \cdot n \\
  &= n^{n-1} (n-1)! \\
  &= n^{n-2} n! \\
  &= S_n \cdot n! \\
\end{aligned}
$$

所以

$$
S_n = n^{n-2}
$$

## 参考资料

[oi-wiki.org](https://oi-wiki.org/graph/prufer/)
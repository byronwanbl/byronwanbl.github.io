---
title: "Kirchhoff 矩阵树定理"
date: 2023-01-11
tags: ["图论", "矩阵树定理", "行列式"]
---

Kirchhoff 定理解决的是对于一个图的生成树的计数问题。

## 无向图

### 定理内容

设 $G$ 为有 $n$ 个顶点的有向图，允许重边，不允许自环。

设 $D(i)$ 为点 $i$ 的度数，$E(u, v)$ 为 $u$ 和 $v$ 之间边的个数。

那么，Kirchhoff 矩阵 $L$ 为：

$$
L(G)_{i, j} =
\begin{cases}
D(i) \ , \text{if}\ i = j \\
-E(i, j) \ , \text{otherwise}
\end{cases}
$$

把矩阵 $L(G)$ 任意删去一行一列，得到 $L'(G)$ ，即：

$$
L'(G) = L(G)
\begin{pmatrix}
1 & 2 & \cdots & i - 1 & i + 1 & \cdots & n \\
1 & 2 & \cdots & i - 1 & i + 1 & \cdots & n
\end{pmatrix}
$$

图 $G$ 生成树的个数为

$$
t(G) = \det(L'(G))
$$

## 有向图

暂不考虑

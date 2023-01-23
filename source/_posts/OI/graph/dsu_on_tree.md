---
title: "树上启发式合并"
date: 2022-07-31
tags: ["图论", "树上启发式合并"]
---

## 适用范围

树上的问题，对于一个节点，需要统计其的子树，且由于一些限制（主要是空间），无法直接合并多个直接子儿子的状态，只能 $\operatorname O (n^2)$ 统计，而 DSU on tree 可以优化至 $\operatorname O (n\cdot log n)$

## 做法

1. 树链剖分
2. 递归处理轻儿子，并删除贡献
3. 递归处理重儿子，并保留贡献
4. 计算轻儿子的贡献

## 代码

```cpp
void dfs(int u, int f) // 树链剖分
{
    ...
}

void dsu(int u, int save /* 是否保留贡献 */)
{
    for (v <- edges(u))
        if (!heavy_son(v)) dsu(v, false);
    dsu(heavy_son(u), true);
    
    for (v <- all_son(u))
        if (v !in heavy_son(u)) add(v);
    
    res(u) = res(u) + calc()

    if (!save)
        for (v <- all_son(u))
            if (v !in heavy_son(u)) del(v);
}
```

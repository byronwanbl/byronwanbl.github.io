---
title: "[CodeForces] 311B 题解"
date: 2022-07-10
---

## 链接

problem: [https://codeforces.com/problemset/problem/311/B](https://codeforces.com/problemset/problem/311/B)

status: [https://codeforces.com/problemset/submission/311/163441918](https://codeforces.com/problemset/submission/311/163441918)

## 题解

$$
dis_i = \sum_{j = 1}^i D_j \newline
time = \{sort(T_i - dis_{H_i})\} \newline
pretime = \sum{j = 1}^i time_i \newline
$$

$$
f_i = f_j + (i - j) * time_i - (pretime_i - pretime_j) \newline
\newline
g_i = g_j + (i - j) * time_i \newline
res = g_i - \sum time_i \newline
$$

$f$ $g$ 两种不同的解法, 有时候只有 $g$ 能够使用 (如以时间为 $i$) .

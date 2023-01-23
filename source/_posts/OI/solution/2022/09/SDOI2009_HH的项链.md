---
title: "[SDOI2009] HH的项链"
date: 2022-09-12
tags: ["离线", "树状数组"]
---

## 大意

给定一个序列，每次询问区间内有多少不同的数。

## 题解

考虑离线处理询问。

先把所有的询问按照右端点从左到右排序，然后考虑加点，查询左端点。

记 $f(x)$ 为当前区间 $x \ldots r$ 的答案

每次加入一个新的数字，
只会对在该数字上一次出现的位置后产生贡献，
即 $f(lst(x) \ldots x) \leftarrow f(lst(x) \ldots x) + 1$ ，
例如：

```text
2 1 5 3 4 2 3 5
    |---------|
        + 1
```

然后用树状数组维护区间修改，单点查询即可。

## 代码

```c++
/*
author: byronwan
problem:
    url: https://www.luogu.com.cn/problem/P1972
    title: [SDOI2009] HH的项链
date: 2022-09-12
*/

#include <bits/stdc++.h>

using namespace std;

const int MAXN = 1e6 + 10;

struct BinaryArray {
    int a[MAXN];

    void update(int i, int x)
    {
        for (; i >= 1; i -= i & (-i)) a[i] += x;
    }

    int query(int i)
    {
        int res = 0;
        for (; i < MAXN; i += i & (-i)) res += a[i];
        return res;
    }
} bin;

int n, m, a[MAXN];
struct Ask {
    int id, l, r;
} ask[MAXN];
int res[MAXN];

void setup()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];

    cin >> m;
    for (int i = 1; i <= m; i++) {
        cin >> ask[i].l >> ask[i].r;
        ask[i].id = i;
    }
    sort(ask + 1, ask + 1 + m, [](auto x, auto y) { return x.r < y.r; });
}

int lst[MAXN];
void pre()
{
    static int lst_i[MAXN];
    for (int i = 1; i <= n; i++) {
        lst[i] = lst_i[a[i]];
        lst_i[a[i]] = i;
    }
}

void solve()
{
    for (int i = 1; i <= m; i++) {
        auto a = ask[i];
        for (int j = ask[i - 1].r + 1; j <= a.r; j++) {
            bin.update(j, 1);
            bin.update(lst[j], -1);
        }
        res[a.id] = bin.query(a.l);
    }
}

int main()
{
    setup();
    pre();
    solve();

    for (int i = 1; i <= m; i++) cout << res[i] << endl;
}
```

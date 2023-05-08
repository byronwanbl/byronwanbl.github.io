---
title: "FFT 快速傅里叶变换"
date: 2023-01-10
tags: ["数学", "多项式", "FFT"]
---
duo xiang shi
本文仅讨论 FFT 在 OI 中的使用。

表述可能不甚严谨，如有错漏，欢迎指出。

## 能解决什么问题？

有两个多项式 $f(x) = a_0 + a_1x^1 + a_2x^2 + \cdots + a_kx^k$， $g(x) = b_0 + b_1x^1 + b_2x^2 + \cdots + b_lx^l$。

求 $h(x) = f(x) \cdot g(x)$。

显然使用朴素算法，时间复杂度是 $\Omicron(kl)$。

但 FFT 可以将其优化到 $\Omicron(n \log n)$ 级别。

## 引理 I （插值）

有 $n$ 个 2元组，为 $(x_0, y_0), (x_1, y_1), \cdots , (x_{n - 1}, y_{n - 1})$ 有 $\forall i \not= j, x_i \not= x_j$，
存在一个唯一的 $n - 1$次多项式，使得 $\forall i \in [0, n)\ f(x_i) = y_i$。

### 唯一性

若有 $f(x), g(x)$ 满足以上条件，设 $r(x) = f(x) - g(x)$。

则 $\forall i \in [0, n)\ r(x_i) = 0$，即 $r$ 有 $n$ 个根。

且因为 $n - 1$ 次方程最多只有 $n - 1$ 个根。

故 $r(x) = 0$

### 存在性

$f(x)$ 可以通过以下方式构造：

$$
f(x) = \sum_{i = 0}^{n - 1} y_i \cdot (\prod_{j \not= i} \frac {x - x_j} {x_i - x_j})
$$

解释：对于所有的 $i$，若 $x = x_i$ 时，对应乘积的部分每一项均为 $1$，否则会有一项为 $0$。

显然因为 $x_i, y_i$ 均为已知量，并且乘积中只有 $n - 1$ 个 $x - x_j$ 相乘，故 $f(x)$ 为 $n - 1$ 次多项式，符合要求。

## 具体步骤

由于 引理I，只要有 $n$ 个不同的 $x$ 和对应的 $f(x)$ ，那么就可以确定 $f(x)$，故我们只需要找出 $h(x)$ 的 $n$ 个互不相同的参数和对应的取值即可。

为了方便，我们设 $n = k + l - 1$。

I. 选取 $x_0, x_1, \dots x_{n-1}$；

II. 确定 $f(x_0), f(x_1), \ldots f(x_{n-1})$ 和 $g(x_0), g(x_1), \ldots g(x_{n-1})$；

III. 根据以上，得出 $h(x_i) = f(x_i) \cdot g(x_i)$，求出 $h$。

乍看时间复杂度还是 $\Omicron(n^2)$ 的，但是不要着急，我们慢慢来。

## 步骤 I

对于 $n - 1$ 次多项式，我们选取
$$
x_k = \omega_n^k = e^{\frac {2 \pi k} n i} = \cos{\frac {2\pi} n} + i \sin{\frac {2\pi} n}
$$

其中 $\omega_n^i$ 为 $x^n = 1$ 的所有根，即 $n$ 次单位虚根。

假如放到复平面上理解，就是单位圆上的 $n$ 等分点。

例如 $n = 8$：

![1](./1.png)

为什么选取它呢？因为有一个性质：折半定理。

$$
\forall n, k \in \mathbb{Z}, \omega_n^k = \omega_{2n}^{2k}
$$

从直觉上来说比较显然，故不作证明。

那么这个有什么用呢？当然有用，我们可以借此折半问题规模，继续往下看。

## 步骤 II (DFT)

因为求 $f(x_i)$ 与求 $g(x_i)$ 本质相同，故仅以 $f(x_i)$ 为例。

为了叙述和编写代码的方便，我们令 $n$ 为总是可以表示成 $n = 2^q$ 。（多出的项补 $0$ 即可，此处 $n$ 与上面不同）

同时，用下标表示序列的长度， $f_m(x_i), m = 2^q$ ，设 $a_{m, i}$ 为 $f_m$ 各项的系数。

求 $f_n(x_i), i \in [0, n)$ 依然是 $\Omicron(n^2)$ 的，要优化到 $\Omicron(n \log n)$ 。

考虑求 $f_n(x_i)$ 的情况：

若 $n = 1$，因为 $\omega_1 = 1$，故 $f_1 = a_{1, 0}$

否则，$n$ 为偶数，所以有：（根据上面的约定 $f_n(x)$ 为 $n-1$ 次多项式）

将 $f_n(x)$ 根据奇偶性拆开：

$$
\begin{aligned}
f_n(x) &= a_{n, 0} + a_{n, 1}x^1 + \cdots + a_{n, n - 1}x^{n - 1} \\
&= (a_{n, 0} + a_{n, 2}x^2 + \cdots + a_{n, n - 2}x^{n - 2}) \\
&\ \ + (a_{n, 1}x^1 + a_{n, 3}x^3 + \cdots + a_{n, n - 1}x^{n - 1}) \\
&= (a_{n, 0}(x^2)^0 + a_{n, 2}(x^2)^1 + \cdots + a_{n, n - 2}(x ^ 2)^{\frac {n - 2} 2}) \\
&\ \ + x(a_{n, 1}(x^2)^0 + a_{n, 3}(x^2)^1 + \cdots + a_{n, n - 1}(x ^ 2)^{\frac {n - 2} 2}) \\
\end{aligned}
$$

把平方提出来，得到：

$$
f_{n, 0}(x) = a_{n, 0}x^0 + a_{n, 2}x^1 + \cdots + a_{n, n - 2}x^{\frac {n - 2} 2} \\
f_{n, 1}(x) = a_{n, 1}x^0 + a_{n, 3}x^1 + \cdots + a_{n, n - 1}x^{\frac {n - 2} 2} \\
$$

那么：

$$
f_n(x) = f_{n, 0}(x ^ 2) + xf_{n, 1}(x ^ 2) \\

f_n(\omega_n^i) = f_{n, 0}(\omega_n^{2i}) + \omega_n^i f_{n, 1}(\omega_n^{2i})
$$

发现，可以引用折半定理，令 $p = \frac n 2$：

$$
f_n(\omega_n^i) = f_{n, 0}(\omega_p^i) + \omega_n^i f_{n, 1}(\omega_p^i)
$$

我们可以观察一下这一番操作的结果：

|  带入的值  |     $0$     |     $1$     | $\cdots$ |     $p - 1$     |     $p + 0$     |     $p + 1$     |     $\cdots$     |     $2p - 1$      |
| ------------- | ----------- | ----------- | -------- | --------------- | --------------- | --------------- | ----------------- | ----------------- |
| $f_n(x)$      | $\omega_n^0$ | $\omega_n^1$ | $\cdots$ | $\omega_n^{p-1}$ | $\omega_n^{p+0}$ | $\omega_n^{p+1}$ | $\cdots$ | $\omega_n^{p+p-1}$ |
| $f_{n, 0}(x)$ | $\omega_p^0$ | $\omega_p^1$ | $\cdots$ | $\omega_p^{p-1}$ | $\omega_p^{p+0}$ | $\omega_p^{p+1}$ | $\cdots$ | $\omega_p^{p+p-1}$ |
| $f_{n, 1}(x)$ | $\omega_p^0$ | $\omega_p^1$ | $\cdots$ | $\omega_p^{p-1}$ | $\omega_p^{p+0}$ | $\omega_p^{p+1}$ | $\cdots$ | $\omega_p^{p+p-1}$ |

可以发现，由于显然有 $\omega_p^i = \omega_p^{i+p}$，
每次只用处理 $f_{n,0}$ 带入 $i\in [0, p), \omega_p^i$ 的值即可，$f_{n, 1}$ 同理。

由于 $p = \frac n 2$，所以每次处理的规模折半，可以实现 $O(n \log n)$ 。

以下是代码：

```cpp
typedef double db;
typedef complex<double> comp;

void fft(int n, comp* a /* 输入数组 */ , int op /* 会有其他的用途，这里暂时忽略，可以认为总为 1 */)
{
    if (n == 1) return;
    comp a0[n >> 1], a1[n >> 1];
    for (int i = 0; i < n >> 1; i++) a0[i] = a[i << 1], a1[i] = a[i << 1 | 1]; // 拆成两半
    fft(n >> 1, a0, op), fft(n >> 1, a1, op); // 分治

    comp wn = comp(cos(2.0 * PI / n), op * sin(2.0 * PI / n)), w = comp(1, 0);
    for (int i = 0; i < (n >> 1); i++, w *= wn) { // 合起来
        a[i] = a0[i] + w * a1[i]; // 前一半
        a[i + (n >> 1)] = a0[i] - w * a1[i]; // 后一半
    }
}
```

## 步骤 III (IDFT)

将 $f$ 和 $g$ 插值后相乘，现在我们需要根据这个相乘的结果反退出 $h(x)$ 的系数，即点值。

可以从矩阵的角度考虑，插值过程中发生了什么。

令

$$
\vec A =
    \begin{bmatrix}
    a_{n, 0} \\
    a_{n, 1} \\
    \vdots \\
    a_{n, n-1} \\
    \end{bmatrix} \\
\ \\

V =
    \begin {bmatrix}
    \omega_n^{0\cdot0} & \omega_n^{0\cdot1} & \cdots & \omega_n^{0\cdot(n-1)} \\
    \omega_n^{1\cdot0} & \omega_n^{1\cdot1} & \cdots & \omega_n^{1\cdot(n-1)} \\
    \vdots & \vdots & \ddots & \vdots \\
    \omega_n^{(n-1)\cdot0} & \omega_n^{(n-1)\cdot1} & \cdots & \omega_n^{(n-1)\cdot(n-1)} \\
    \end{bmatrix} \\
\ \\

\vec Y =
    \begin{bmatrix}
    f_n(\omega_n^0)\\
    f_n(\omega_n^1)\\
    \vdots \\
    f_n(\omega_n^{n-1})\\
    \end{bmatrix} = V \vec A \\
$$

现在对于 $h$ 知道了 $\vec Y$ 和 $V$，求 $\vec A$ ，
显然 $\vec A = \vec Y V^{-1}$ 。

令
$$
U =
    \begin {bmatrix}
    \omega_n^{-0\cdot0} & \omega_n^{-0\cdot1} & \cdots & \omega_n^{-0\cdot(n-1)} \\
    \omega_n^{-1\cdot0} & \omega_n^{-1\cdot1} & \cdots & \omega_n^{-1\cdot(n-1)} \\
    \vdots & \vdots & \ddots & \vdots \\
    \omega_n^{-(n-1)\cdot0} & \omega_n^{-(n-1)\cdot1} & \cdots & \omega_n^{-(n-1)\cdot(n-1)} \\
    \end{bmatrix} \\
\ \\

VU =
    \begin {bmatrix}
    n & 0 & 0 & \cdots & 0 \\
    0 & n & 0 & \cdots & 0 \\
    0 & 0 & n & \cdots & 0 \\
    \vdots & \vdots & \vdots & \ddots & \vdots \\
    0 & 0 & 0 & \cdots & n \\
    \end {bmatrix} \\
\ \\
V^{-1} = \frac 1 n U \\
$$

解释：

令 $X_{i, j}$ 为矩阵 $X$ 第 $i$ 行 $j$ 列的值。令 $B = VU$ 。

要证明

$$
B_{i,j} =
    \left\{\begin{aligned}
    n,\ & \text{if}\ i = j \\
    0,\ & \text{otherwise}
    \end{aligned} \right. \\
$$

有

$$
B_{i, j} = \sum_{k=0}^n V_{i,k}U_{k, j}
  = \sum_{k=0}^n \omega^{ik} \omega^{-kj}
  = \sum_{k=0}^n \omega^{k(i - j)}
$$

当 $i = j$ 时，$B_{i, j} = \sum_{k=0}^n \omega^0 = n$ 。

否则设 $t = i - j, t \not= 0$，要证明

$$
B_{i, j} = \sum_{k=0}^n \omega^{kt} = 0
$$

即将其看作复平面的单位圆上 $n$ 个点，第 $0$ 个点为 $(1, 0)$，相邻点间隔 $\frac {2\pi} n t$，求每个点是否都有唯一的点对应使得二者之和为 $0$ （过原点对称，且这里只考虑 $n$ 为 $2^q$ 的情况）。

令 $t = 2^p \cdot c$， $n = 2^q$， $c$ 为奇数，且 $p < q$。

把 $n$ 个点分成 $2^p$ 组，每组 $2^{q-p}$ 个，显然每组相同，故只考虑第 $0$ 组。

对于该组的第 $k \lt \frac {2^{q-p}} 2$ 个，其与第 $k + \frac {2^{q-p}} 2$ 个相对应，因为有

$$
\omega^{k + t \frac {2^ {q-p}} 2}
 = \omega^k \omega^{2^p \cdot c \cdot \frac {2^q} {2^{p + 1}}}
 = \omega^k (\omega^{\frac{2^q} 2})^c
 = \omega^k (\omega^{\frac n 2})^c
 = \omega^k (-1)^c
 = -\omega^k
$$

所以只要把插值过程中的 $\omega_n^i$ 变为 $\omega_n^{-i}$ ，再对结果乘 $\frac 1 n$ 即为答案。

代码 ([luogu P3803](https://www.luogu.com.cn/problem/P3803))：

```cpp
/*
author: byronwan
problem:
    url: https://www.luogu.com.cn/problem/P3803
    title: FFT
date: 2022-11-12
*/

#include <bits/stdc++.h>

using namespace std;

typedef double db;
typedef complex<double> comp;

const int MAXN = 4e6 + 10;
const double PI = acos(-1);

int k, l, n;
comp a[MAXN], b[MAXN];

void setup()
{
    cin >> k >> l;
    for (int i = 0; i <= k; i++) cin >> a[i];
    for (int i = 0; i <= l; i++) cin >> b[i];
    for (n = 1; n <= k + l; n <<= 1)
        ;
}

void fft(int n, comp* a, int op)
{
    if (n == 1) return;
    comp a0[n >> 1], a1[n >> 1];
    for (int i = 0; i < n >> 1; i++) a0[i] = a[i << 1], a1[i] = a[i << 1 | 1];
    fft(n >> 1, a0, op), fft(n >> 1, a1, op);

    comp wn = comp(cos(2.0 * PI / n), op * sin(2.0 * PI / n)), w = comp(1, 0);
    for (int i = 0; i < (n >> 1); i++, w *= wn) {
        a[i] = a0[i] + w * a1[i];
        a[i + (n >> 1)] = a0[i] - w * a1[i];
    }
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr), clog.tie(nullptr);

    setup();
    fft(n, a, 1), fft(n, b, 1);
    
    for (int i = 0; i < n; i++) a[i] = a[i] * b[i];
    fft(n, a, -1);

    for (int i = 0; i <= k + l; i++) cout << (long long)(a[i].real() / n + 0.5) << " ";
    cout << endl;
}
```

## 迭代实现

刚才的代码是递归实现，由于一些原因(个人猜测是要在栈上开数组)，递归实现虽然写起来方便，但可能会被卡，以下介绍不递归的迭代实现。

首先，我们以 $n = 8$ 为例，模拟递归的过程。

```plain
               a0 a1 a2 a3 a4 a5 a6 a7
              a0 a2 a4 a6 | a1 a3 a5 a7
            a0 a4 | a2 a6 | a1 a5 | a3 a7
        a0 | a4 | a2 | a6 | a1 | a5 | a3 | a7

binary  000  100  010  110  001  101  011  111
```

不难发现，每一位上 `a` 的下标是该位的下标的二进制反过来。

我们首先把 `a` 按照这个规律放好（跳过了递归实现中分配 `a` 的步骤），直接开始合并，而合并的方法和原来相同（合并过程中不需要 `a`，只要位置对了就行）。

那么问题就是如何把 `a` 放好，设 $rev(i)$ 为 $i$ 二进制反过来的数，显然有 $rev(rev(i)) = i$，所以 $rev(i)$ 和 $i$ 是一一映射的。

考虑用递推的方式求 $rev(i)$。

```plain
     i: a b c d e  f
        ^---+---^  ^
            |      |
            +--+   |
               |   |
        +------|---+
        |      |
        +  +---+---+
rev(i): f  e d c b a
```

所以

```cpp
rev[i] = 
    (rev[i >> 1] >> 1 /* 除以2是因为它的长度为 log n 原来的最高位即现在的最低位为 0 */)
    | ((i & 1) << (l - 1 /*长度*/))
```

代码：

```cpp
/*
author: byronwan
problem:
    url: https://www.luogu.com.cn/problem/P3803
    title: FFT
date: 2023-01-10
*/

#include <bits/stdc++.h>

using namespace std;

typedef double db;
typedef complex<double> comp;

const int MAXN = 4e6 + 10;
const double PI = acos(-1);

int k, l, n, m;
int rev[MAXN];
comp a[MAXN], b[MAXN];

void setup()
{
    cin >> k >> l;
    for (int i = 0; i <= k; i++) cin >> a[i];
    for (int i = 0; i <= l; i++) cin >> b[i];
    for (n = 1; n <= k + l; n <<= 1, m++)
        ;
}

void fft(comp* a, int op)
{
    for (int i = 0; i < n; i++) {
        rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (m - 1));
        if (rev[i] > i) swap(a[rev[i]], a[i]);
    }

    for (int p = 1; p < n; p <<= 1) {
        int len = p * 2;
        auto wn = comp(cos(2.0 * PI / len), op * sin(2.0 * PI / len));
        for (int i = 0; i < n; i += len) {
            auto w = comp(1, 0);
            for (int j = 0; j < p; j++, w *= wn) {
                auto x = a[i + j], y = w * a[i + j + p];
                a[i + j] = x + y, a[i + j + p] = x - y;
            }
        }
    }
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr), clog.tie(nullptr);

    setup();
    fft(a, 1), fft(b, 1);

    for (int i = 0; i < n; i++) a[i] = a[i] * b[i];
    fft(a, -1);

    for (int i = 0; i <= k + l; i++) cout << (long long) (a[i].real() / n + 0.5) << " ";
    cout << endl;
}
```

## 参考资料

[「自为风月马前卒」大大的博客](https://www.cnblogs.com/zwfymqz/p/8244902.html)

[oi-wiki.org](https://oi-wiki.org/math/poly/fft/)

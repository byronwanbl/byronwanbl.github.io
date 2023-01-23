---
title: "NTT 数论变换"
date: 2023-01-11
tags: ["数学", "多项式", "NTT"]
---

在 FFT 中，需要用到复数和 double，这有两个问题：精度较低和无法实现取模。

所幸，这两个问题都可以用一个东西解决：NTT。

NTT 所解决的问题和 FFT 相似：

有两个多项式 $f(x) = a_0 + a_1x^1 + a_2x^2 + \cdots + a_kx^k$， $g(x) = b_0 + b_1x^1 + b_2x^2 + \cdots + b_lx^l$。

求 $h(x) = f(x) \cdot g(x)$ ，但 $h(x)$ 每项系数对 $P$ 取模。

我们考虑用一个叫做原根的东西代替单位根，设 $n$ 次单位根为 $\omega_n$ 。

## 四条性质

在 FFT 中所用到了四条单位根的性质，它们是：

1. 对于 $0 \le i \lt n$ ， $\omega_n^i$ 各不相同。

2. 折半定理 $\omega_n^i$ = $\omega_{2n}^{2i}$ ，用于分治。

3. $\omega_n^i$ = -$\omega_n^{i + \frac n 2}$ ，用于分治。

4. 对于 $t = \sum_{i=0}^{n-1} \omega_n^{ki}$ ， 当 $k = 0$ 时， $t = n$ ，否则 $t = 0$ ，用于 IDFT 。

对于质数 $P = 2^t q + 1$ ，其的原根 $g$ 为满足 $g^i, 0 \le i \lt P$ 互不相同的数。

考虑这四个性质，令 $n = 2^t$ ， $\omega_n = g^q$ ：

1. 显然 $0 \le 0, q, 2q, \cdots , (n-1)q \lt P$ ，故 $\omega^0, \omega^1, \omega^2, \cdots, \omega^{n-1} \lt P$ 互不相同，满足。

2. $\omega_{2n}^{2i} = (g^{\frac n 2})^{2i} = g^{ni} = \omega_n^i$

3. 根据费马小定理 $\omega^n = g^{P-1} = 1\ (\bmod\ P)$，又因为 $(\omega^{\frac n 2})^2 = \omega^n = 1$ ，所以 $\omega^{\frac n 2} = 1 \text{ or } -1$ ，又因为 $g^i$ 互不相同，所以 $\omega^{\frac n 2} = -1$ ，所以 $\omega_n^i = -\omega_n^{i + \frac n 2}$

4. 当 $k \not= 0$ 时

$$
\begin{aligned}
S(k) &= (\omega^k)^0 + (\omega^k)^1 + \cdots + (\omega^k)^{(n-1)} \\
\omega^k S(k) &= (\omega^k)^1 + (\omega^k)^2 + \cdots + (\omega^k)^n \\
\omega^k S(k) - S(k) &= (\omega^k)^n - 1 \\
S(k) &= \frac {\omega^{nk} - 1} {\omega^k - 1}
\end{aligned}
$$

由于 $\omega_n \bmod P = 1$ ，所以 $\omega^{nk} - 1 \bmod P = 0$ ，所以 $S(k) = 0$ 。

## 求原根

先略过，记住几个常用的模数的原根。

| $P$ | 分解 | $g$ |
|-----|-----|-----|
| $998244353$ | $7 \times 17 \times 2^{23} + 1$ | $3$ |
| $1004535809$ | $479 \times 2^{21} + 1$ | $3$ |

## 参考资料

[oi-wiki.org](https://oi-wiki.org/math/poly/ntt/)

[「Menci」大大的博客](https://oi.men.ci/fft-to-ntt/)

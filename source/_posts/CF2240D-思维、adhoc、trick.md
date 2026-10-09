---
title: CF2240D 思维、adhoc、trick
date: 2026-08-11 12:08:34
categories: [题解]
tags: [Codeforces, 思维]
---
[Problem - D - Codeforces](https://codeforces.com/contest/2240/problem/D)
我们任选两个互相在视野内的 $i,j$ 那么我们假定选了 $i$,
则我们可以先行加上贡献 $a_i - a_j$ ，假如我们选择了 $j$ 之后，贡献会变成 $a_i - a_j + a_j - a_i = 0$ ,我们发现对于两个变量 其贡献为 $S_i*(a_i-a_j)+S_j*(a_j-a_i)$ ，我们发现两个选的贡献是完全分开的，也就是说选了 $i$ 这个值，一定就有固定贡献 
$\sum_{j\in V_i}(a_i-a_j) = 2d*a_i-\sum_{j\in V_i}a_j$ 所以选还是不选 只看该值为正还是为负

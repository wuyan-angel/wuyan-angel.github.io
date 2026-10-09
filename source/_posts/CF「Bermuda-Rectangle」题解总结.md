---
title: CF「Bermuda Rectangle」题解总结
date: 2026-08-27 00:27:32
categories: [题解]
tags: [Codeforces]
---
# CF「Bermuda Rectangle」题解总结

[Submission #388487117 - Codeforces](https://codeforces.com/contest/2257/submission/388487117)
## 题意

定义"百慕大矩形"：面积恰好为 $S$、边长为正整数 $(a,b)$（$a\cdot b=S$）、左下角在 $(0,0)$ 的矩形。

给出 $S$ 和 $q$ 个查询 $(x,y)$，每次求：

> 所有百慕大矩形**并集** $F$ 与查询矩形 $[0,x)\times[0,y)$ 的交集面积。

---

## 核心思路

### 1. 把所有矩形变成"阶梯形"

每个合法矩形由右上角 $(a,b)$ 决定，其中 $a$ 取遍 $S$ 的因子，$b=S/a$。

把这些矩形画出来，它们的并集 $F$ 是一个**单调下降的阶梯形**。例如 $S=20$：

```
高
20|█████
10|█████░░
 5|████████░░
 4|█████████░
 2|██████████████░░
 1|████████████████████
  +----+----+----+----+----+----
  0    1    2    4    5   10   20
```

因子升序为 $d=[1,2,4,5,10,20]$，台阶高度为 $h_i = S/d_i$（从大到小）。

### 2. 预处理：因子 + 前缀面积

- 用 $O(\sqrt S)$ 枚举求出所有因子，排序。
- 设 `pre[i]` = $F$ 在 $[0, d_i)$ 的面积，递推：

$$pre[i] = pre[i-1] + (d_i - d_{i-1})\cdot\frac{S}{d_i}$$

### 3. 查询：二分找分界点

水平线 $y$ 把阶梯切成两段：

- 高度 $\ge y$ 的台阶 → 贡献**满高** $y$；
- 高度 $< y$ 的台阶 → 贡献台阶**自身高度**。

因为高度单调下降，用 `upper_bound` 找**第一个高度 $<y$ 的台阶**：

$$\frac{S}{d_i}<y \iff d_i>\frac{S}{y}$$

设 `r` 为最后一个"满高"台阶的下标（即 `d[r]` 是满高区右端点）。

- 若 $x \le d[r]$：整个查询区都满高，答案 $= x\cdot y$；
- 否则：
  - 满高区面积 $= d[r]\cdot y$；
  - 阶梯区面积 $= P(x) - pre[r]$，其中 $P(x)$ 是 $F$ 在 $[0,x)$ 的面积，用 `lower_bound(x)` + 前缀和算。

---

## 完整代码（已修复 $x>S$ 的越界）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int T;
    cin >> T;
    while (T--) {
        ll s;
        int q;
        cin >> s >> q;

        // 1) 求所有因子，升序（d[0]=0 作哨兵）
        vector<ll> d(1, 0);
        for (ll i = 1; i * i <= s; i++) {
            if (s % i == 0) {
                d.push_back(i);
                if (i * i != s) d.push_back(s / i);
            }
        }
        sort(d.begin() + 1, d.end());
        int k = (int)d.size() - 1;

        // 2) 前缀面积 pre[i] = F 在 [0, d[i]) 的面积
        vector<ll> pre(k + 1, 0);
        for (int i = 1; i <= k; i++)
            pre[i] = pre[i - 1] + (d[i] - d[i - 1]) * (s / d[i]);

        // 3) 查询
        while (q--) {
            ll x, y;
            cin >> x >> y;

            // 关键：F 在 x>s 处高度为 0，截断防止 lower_bound 越界
            x = min(x, s);

            // r = 最后一个"满高"(高度>=y) 台阶的下标
            int r = upper_bound(d.begin() + 1, d.end(), s / y)
                    - d.begin() - 1;

            if (x <= d[r]) {
                cout << x * y << '\n';
                continue;
            }

            // P(x) = F 在 [0, x) 的面积
            int p = lower_bound(d.begin() + 1, d.end(), x) - d.begin();
            ll fx = pre[p - 1] + (x - d[p - 1]) * (s / d[p]);

            // 满高区 + 阶梯区
            ll ans = d[r] * y + (fx - pre[r]);
            cout << ans << '\n';
        }
    }
    return 0;
}
```

---

## 复杂度

- **预处理**：$O(\sqrt S)$（枚举因子）+ $O(k\log k)$（排序），$k$ 为因子个数。
- **每个查询**：两次二分，$O(\log S)$。
- **总复杂度**：$O(\sqrt S + q\log S)$。

---

## 易错点

1. **$x>S$ 必须截断**（`x = min(x, s)`），否则 `lower_bound` 返回 `end`，访问 `d[p]` 越界/除零。
2. **数组用 `vector` 动态分配**，避免因子数超过硬编码上限。
3. 若 $S$ 很大，`x*y` 可能溢出 `long long`，可用 `__int128` 或也截断 `y = min(y, s)` 缓解。

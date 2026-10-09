---
title: ARC229_B 思维+思维转换
date: 2026-09-29 11:25:59
categories: [题解]
tags: [AtCoder, 思维]
---

> 🎯 **一句话题意**：给定数组 A 和操作（位置 i 减 v、i+1 减 ⌊v/2⌋），求把 A 清零的最少操作次数，不可行输出 -1。

> 🔗 **原题链接**：[B - Halving Subtraction](https://atcoder.jp/contests/arc229/tasks/arc229_b?lang=en)
### 核心 Trick 总结

1. **单次扣减的线性关系**：操作数值 $x$ 在位置 $i$ 减去 $v$，在位置 $i+1$ 减去 $\lfloor v/2 \rfloor$。由 $v = 2\lfloor v/2 \rfloor + (v \bmod 2)$ 可知，位置 $i$ 比 $2 \times$ 位置 $i+1$ 恰好**多减去了一个二进制位 $(0 \text{ 或 } 1)$**。
2. **差值降维（$d_i$）**：$k$ 次操作累加后，必须满足 $0 \le A_i - 2A_{i+1} \le k$。令 $d_i = A_i - 2A_{i+1}$，则 $d_i$ 就是第 $i-1$ 个二进制位上需要的 `1` 的总个数。若存在 $d_i < 0$，直接输出 `-1`。
3. **末尾 $A_N$ 的自由注入**：末尾 $A_N$ **完全不卡操作次数**。只要操作次数 $k \ge 1$，直接在任意一个选出的数字（如 $x_1$）上加上高位值 $A_N \cdot 2^{N-1}$。它在第 $i$ 个位置产生的扣减量恰好是 $A_N \cdot 2^{N-i}$，**精确对冲掉 $A_i$ 中由 $A_N$ 带来的基础骨架，且对所有 $d_i$ 零钱位零干扰**。
4. **按位独立与最少次数**：二进制各个数位互不干扰，因此最少操作次数就是所有数位最大需求量：$\max(d_1, d_2, \dots, d_{N-1})$（非全 0 数组至少为 1）。

---

### 解题思路与构造逻辑

#### 一、 数组 $A$ 的结构拆解

将递推关系 $d_i = A_i - 2A_{i+1}$ 展开，可以发现数组 $A_i$ 本身就是由末尾 $A_N$ 与散落零钱 $d$ 组成的：


$$A_i = A_N \cdot 2^{N-i} + \sum_{j=i}^{N-1} d_j \cdot 2^{j-i}$$

* **项一（$A_N \cdot 2^{N-i}$）**：来自末尾 $A_N$ 的高位放大骨架。
* **项二（$\sum d_j \cdot 2^{j-i}$）**：来自各个二进制位缺口 $d_j$ 的低位补丁。

---

#### 二、 完美对冲的构造方案

假设通过各数位缺口计算出的最少操作次数为 $k = \max(1, \max d_i)$（非全 0 数组）：

1. **消去 $A_N$ 的骨架**：
直接令第一个操作数 $x_1$ 的高位加上 $A_N \cdot 2^{N-1}$。
在第 $i$ 个位置，该项产生的扣减量为：

$$\lfloor (A_N \cdot 2^{N-1}) / 2^{i-1} \rfloor = A_N \cdot 2^{N-i}$$



**这一项直接精确抹平了 $A_i$ 中所有由 $A_N$ 带来的数值！**
2. **消去 $d_i$ 的补丁**：
对每个数位缺口 $d_i$，我们在 $k$ 个操作数的第 $i-1$ 个二进制位上，挑选 $d_i$ 个数字填入 `1`。由于 $k \ge d_i$，这 $k$ 个行空间必定够装。

最终构成的 $k$ 个操作数，即可在 $1$ 次高位注入 + $k$ 次低位拼装下，**无残渣、无超减地将整张数组 $A$ 完美清零**。

---

### AC 代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;
typedef long long ll;

void solve() {
    int n;
    cin >> n;
    vector<ll> a(n + 1);
    bool all_zero = true;

    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        if (a[i] > 0) all_zero = false;
    }

    if (all_zero) {
        cout << 0 << "\n";
        return;
    }

    // 数组非全 0 时，至少需要 1 次操作（用于承载 A_N 的高位注入）
    ll ans = 1;
    for (int i = 1; i < n; i++) {
        ll d = a[i] - 2 * a[i + 1];
        if (d < 0) { // 出现了不可能满足的负差值
            cout << -1 << "\n";
            return;
        }
        ans = max(ans, d);
    }

    cout << ans << "\n";
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}

```

---

### 复杂度分析

* **时间复杂度**：$O(N)$，对输入数组做一次线性扫描计算 $a[i] - 2a[i+1]$，在 $T \le 10^4, N \le 30$ 下只需 $3 \times 10^5$ 次计算，轻松通过。
* **空间复杂度**：$O(N)$，只需存储原数组 $A$。

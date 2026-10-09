---
title: 分数规划好题 + 思维转换 abc469_e
date: 2026-07-20 12:06:00
categories: [题解]
tags: [AtCoder, 思维]
---
[E - Pro Exam Eligibility](https://atcoder.jp/contests/abc469/tasks/abc469_e?lang=en)
## 【题解】Pro Exam Eligibility

---

### 题目大意

给定一个长度为 $N$ 的 01 字符串 $S$（`o` 表示胜，`x` 表示负），寻找一个区间 $[l, r]$，满足：

1. 区间内 `o` 的数量 $\ge K$；
2. 最大化胜率 $\frac{\text{cnt}_o(l, r)}{r - l + 1}$。

---

### 核心 Trick 拆解

这道题把三大竞赛常用技巧嵌套在一起：

1. **Trick 1：0-1 分数规划（二分答案）**
只要遇到“最大化平均值 / 胜率 / 比例”且带区间约束的问题，第一反应就是**二分胜率 $x$**，将“求最大值”转化为“判定胜率 $x$ 是否可行”。
2. **Trick 2：权值转换（将比例转化成区间和）**
判定条件 $\frac{\text{cnt}_o(l, r)}{r - l + 1} \ge x$ 等价于：

$$\text{cnt}_o(l, r) - x \cdot (r - l + 1) \ge 0$$



将 `'o'` 的权值设为 $1 - x$，`'x'` 的权值设为 $-x$，问题转换为：是否存在一个合法区间的权值和 $\ge 0$。
3. **Trick 3：双指针 + 前缀最小值 $O(N)$ 检查**
由于 'o' 的数量前缀和 $C$ 是单调递增的，满足 $\text{cnt}_o(l, r) \ge K$ 的最大合法左端点减一（记为 $ptr$）随右端点 $r$ 的右移而**单调不减**。结合前缀最小值数组 $M$，可以在 $O(N)$ 时间内判断是否存在合法的 $l$ 使得 $P_r - P_{l-1} \ge 0$。

---

### 详细推导过程

#### 1. 转化为判定性问题

假设当前二分测试的胜率为 $x$：

* 设权值数组 $w_i = \begin{cases} 1 - x, & S[i] = \text{'o'} \\ -x, & S[i] = \text{'x'} \end{cases}$
* 设权值前缀和 $P_i = \sum_{j=1}^i w_j$（规定 $P_0 = 0$）
* 设 'o' 数量前缀和 $C_i = \sum_{j=1}^i [S[j] == \text{'o'}]$

区间 $[l, r]$ 的权值和为 $P_r - P_{l-1}$。若存在某个 $r$ 及合法的 $l$，使得：


$$P_r - P_{l-1} \ge 0$$


则说明胜率 $x$ 是可达到的。

#### 2. 区间合法性与双指针优化

对于固定的右端点 $r$，左端点 $l$ 必须满足：


$$C_r - C_{l-1} \ge K \iff C_{l-1} \le C_r - K$$

因为 $C$ 数组单调不减，所以满足 $C_j \le C_r - K$ 的下标 $j$ 必然是一个连续前缀 $[0, ptr_r]$。
随着 $r$ 的增大，$C_r - K$ 也在增大，因此 $ptr_r$ 具有**单调性**，可用双指针维护。

#### 3. 极值快速查询

对于右端点 $r$，我们要寻找是否存在 $j \in [0, ptr_r]$ 使得 $P_r - P_j \ge 0$。
要让 $P_r - P_j$ 尽可能大，我们只需取最小值 $P_j$。
定义前缀最小值数组：


$$M_k = \min_{0 \le j \le k} P_j$$

检查条件简化为：**是否存在某个 $r \in [1, N]$，满足 $P_r - M_{ptr_r} \ge 0$。**

---

### 复杂度分析

* **时间复杂度**：单次 `check` 仅需遍历一遍字符串，复杂度为 $O(N)$。二分 80 次，浮点数精度可达 $10^{-15}$（远超要求的 $10^{-6}$）。总时间复杂度为 $O(80 \cdot N)$，运行时间约 $0.1$ 秒。
* **空间复杂度**：存储前缀和数组需要 $O(N)$ 空间。

---

### C++ 标准实现

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <iomanip>
#include <algorithm>

using namespace std;

// O(N) 检查当前胜率 x 是否能达到
bool check(double x, int n, int k, const string& s, const vector<int>& C, vector<double>& P, vector<double>& min_P) {
    P[0] = 0.0;
    min_P[0] = 0.0;
    
    // 1. 构建权值前缀和与前缀最小值
    for (int i = 1; i <= n; ++i) {
        double val = (s[i] == 'o') ? (1.0 - x) : (-x);
        P[i] = P[i - 1] + val;
        min_P[i] = min(min_P[i - 1], P[i]);
    }
    
    // 2. 双指针维护满足 C[i] - C[ptr] >= k 的最大下标 ptr (即 l-1 的最大取值)
    int ptr = 0;
    for (int i = 1; i <= n; ++i) {
        while (ptr + 1 <= i && C[i] - C[ptr + 1] >= k) {
            ptr++;
        }
        
        // 3. 判断是否存在合法的左端点使得区间权值和 >= 0
        if (C[i] - C[ptr] >= k) {
            if (P[i] - min_P[ptr] >= -1e-12) { // 考虑浮点数误差
                return true;
            }
        }
    }
    return false;
}

void solve() {
    int n, k;
    if (!(cin >> n >> k)) return;
    string s;
    cin >> s;
    s = " " + s; // 转换为 1-based 下标
    
    // 预处理 'o' 的前缀和
    vector<int> C(n + 1, 0);
    for (int i = 1; i <= n; ++i) {
        C[i] = C[i - 1] + (s[i] == 'o');
    }
    
    // 复用数组，避免多次分配内存
    vector<double> P(n + 1, 0.0);
    vector<double> min_P(n + 1, 0.0);
    
    // 二分胜率 x，范围在 [0, 1]
    double l = 0.0, r = 1.0;
    for (int iter = 0; iter < 80; ++iter) {
        double mid = l + (r - l) / 2.0;
        if (check(mid, n, k, s, C, P, min_P)) {
            l = mid; // 胜率 mid 可行，尝试更高的胜率
        } else {
            r = mid; // 胜率 mid 不可行
        }
    }
    
    cout << fixed << setprecision(15) << l << "\n";
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    solve();
    return 0;
}

```

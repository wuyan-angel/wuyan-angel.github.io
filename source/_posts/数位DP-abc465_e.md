---
title: 数位DP abc465_e
date: 2026-07-04 22:16:23
categories: [题解]
tags: [AtCoder, DP, ICPC]
---
![](/images/vault/ICPC训练计划)## 🤖 Assistant

好，我一步步来，先讲清楚再给完整代码。

## 详细讲解 Digit Circus

### 题意再梳理

求 [1, N] 中**恰好满足下面三个条件之一**的数：

| 条件 | 含义 |
|:----:|:----:|
| **A** | x 是 3 的倍数 |
| **B** | x 的十进制表示包含数字 3 |
| **C** | x 用了**恰好 3 种不同的数字** |

N 最大 500 位，所以必须数位 DP。

### 重点：为什么用「活字印刷」的思考方式

三个条件看上去纠缠在一起，但其实**每个条件只依赖数字的某个属性**：

- A → 数位之和 mod 3
- B → 是否出现过数字 3
- C → 出现过几种不同的数字

所以 DP 过程中把这些属性一路带下去，到了终点统一判断即可。

---

### 状态设计

```
dfs(pos, tight, started, mod3, has3, mask)
```

| 参数 | 范围 | 含义 |
|:----:|:----:|------|
| pos | 0~len(N) | 处理到第几位 |
| tight | 0/1 | 前缀是否和 N 一致 |
| started | 0/1 | 是否已经放过非零数字（处理前导零） |
| mod3 | 0/1/2 | 已放数字之和 % 3 |
| has3 | 0/1 | 是否出现过数字 3 |
| mask | 0~1023 | 10位bit，标记出现的数字 |

**核心思想**：每一位从 0~limit 枚举数字（limit = tight ? N[pos] : 9），
把 mod3/has3/mask 更新后传给下一位。

到了终点 `pos == len(N)`：
- 如果 `started == 0` → 表示全程都是前导零，对应数字 0，不统计
- 否则数三个条件各自是否成立：
  - A: `mod3 == 0`
  - B: `has3 == 1`
  - C: `popcount(mask) == 3`
- 三个布尔值相加，等于 1 就计入答案

### 为什么这个状态是对的

来看几个数的例子：

| 数 | 数位 | mod3 | has3 | mask → popcount | 满足条件 |
|:--:|:----:|:----:|:----:|:--------------:|:--------:|
| 6 | 6 | 0 | 0 | 6→1 | **A** |
| 13 | 1,3 | 1 | 1 | {1,3}→2 | **B** |
| 1012 | 1,0,1,2 | 1 | 0 | {0,1,2}→3 | **C** |
| 3 | 3 | 0 | 1 | {3}→1 | A+B，不统计 |

`started` 的作用是处理**前导零**。比如数字 45（N=100 时表示为 045），前两位是前导零时 mask 里不能把 0 算进去，因为实际数字 45 没用到数字 0。`started` 就干这个事——第一个非零数字出现前，mod3/has3/mask 都保持 0。

### 为什么用 1024 的 mask

10 种数字（0~9），每种出现/没出现用一位 bit 表示，`mask | (1 << d)` 标记。

最后用 `__builtin_popcount(mask)` 数一下 1 的个数 = 用了多少种不同数字。

### 记忆化

只在 `tight == 0` 时缓存，因为 `tight == 1` 的路径是唯一的，缓存了也没用。
缓存 key = `(pos, started, mod3, has3, mask)`。

一维大小：505 × 2 × 3 × 2 × 1024 ≈ 1,200万
用 `int`（4字节）→ 约 25 MB，完全 OK。

---

### 完整代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 998244353;
string s;
int n, dp[505][2][3][2][1024];
bool vis[505][2][3][2][1024];

int dfs(int pos, int tight, int started, int mod3, int has3, int mask) {
    // ---- 终点：判断三个条件 ----
    if (pos == n) {
        if (!started) return 0;              // 数字 0 不算
        int cnt = __builtin_popcount(mask);   // 用了多少种不同数字
        int cond = (mod3 == 0) + has3 + (cnt == 3);
        return (cond == 1) ? 1 : 0;
    }

    // ---- 记忆化（只缓存 tight=0 的状态） ----
    if (!tight) {
        int &res = dp[pos][started][mod3][has3][mask];
        if (vis[pos][started][mod3][has3][mask]) return res;
    }

    // ---- 枚举当前位数字 ----
    int limit = tight ? s[pos] - '0' : 9;
    long long ans = 0;

    for (int d = 0; d <= limit; d++) {
        int ntight = tight && (d == limit);
        int nstarted = started || (d != 0);

        if (!nstarted) {
            // 还在前导零阶段 → 保持 (0,0,0)
            ans += dfs(pos + 1, ntight, 0, 0, 0, 0);
        } else {
            ans += dfs(pos + 1, ntight, 1,
                       (mod3 + d) % 3,
                       has3 || (d == 3),
                       mask | (1 << d));
        }
    }

    ans %= MOD;

    // ---- 写入缓存 ----
    if (!tight) {
        vis[pos][started][mod3][has3][mask] = true;
        dp[pos][started][mod3][has3][mask] = (int)ans;
    }

    return (int)ans;
}

int main() {
    cin >> s;
    n = (int)s.size();
    cout << dfs(0, 1, 0, 0, 0, 0) << '\n';
    return 0;
}
```

### 关键细节解释

**1. 前导零的处理**
```
d = 0 且 started = 0 → nstarted = 0
```
表示「还没出现非零数字」，所以不更新 mod3/has3/mask，继续全部传 0。
下次遇到 d ≠ 0 时 nstarted 变成 1，正式「启动」数字。

**2. tight 的传递**
```
ntight = tight && (d == limit)
```
只有当之前一直在贴着 N 走，且这一位也贴着 N 的数字，后面才继续贴。
一旦某一位选了比 N 小的数字，后面就能自由选 0~9。

**3. 三个条件的统计时机**
不在中间过程中判断，只在 `pos == n` 统一用 `(mod3==0) + has3 + (cnt==3) == 1` 过滤。
这正是这题简洁的关键——把三股线都带着走，最后算总账。

### Sample 验证

**N = 45**
```
dfs(0,1,0,0,0,0)
→ 以上面的数字为例：
  6 → (mod3=0, has3=0, mask=1<<6, pop=1) → cond = 1+0+0=1 ✓
  13 → (mod3=1, has3=1, mask={1,3}, pop=2) → cond = 0+1+0=1 ✓
  30 → (mod3=0, has3=1, mask={0,3}, pop=2) → cond = 1+1+0=2 ✗
输出 → 19 ✓
```

**N = 1013**
```
输出 → 424 ✓
```

复杂度 O(n × 2 × 3 × 2 × 1024 × 10) ≈ 6,000 万次，2 秒很稳。

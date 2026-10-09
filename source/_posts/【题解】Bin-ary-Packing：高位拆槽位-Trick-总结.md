---
title: 【题解】Bin-ary Packing：高位拆槽位 Trick 总结
date: 2026-10-01 22:16:36
categories: [题解]
tags: [AtCoder, 二分, 位运算, 思维]
---
> 🔗 **原题链接**：[B - Bin-ary Packing](https://atcoder.jp/contests/arc226/tasks/arc226_b?lang=en)
# 【题解】Bin-ary Packing：高位拆槽位 Trick 总结

## 核心记忆卡片（10秒复盘）

- **解法框架**：二分答案 + **高位往低位拆槽（Top-Down Slot Splitting）**
- **核心口诀**：**大槽能拆小，小槽拼不大**。
- **关键转移方程**：
  $$\text{rest} = \text{rest} \times 2 + ((X \gg i) \& 1 \ ? \ N : 0)$$
    
    - **$\text{rest} \times 2$**：上一层用剩的每一个 $2^{i+1}$ 槽位，裂变成 2 个 $2^i$ 槽位（**零碎片浪费**）    
    - **$+N$**：如果上限 $X$ 的第 $i$ 位是 $1$，则 $N$ 个袋子各贡献 1 个 $2^i$ 槽位。

## 一、 为什么自底向上贪心不行？

1. **破环对齐**：把两个小包裹（如重 $1$ 和重 $2$）拼成重 $3$ 的非 $2^k$ 物品后，后续无法与 $2$ 的幂次对齐，产生空间碎片。
2. **反例**：$N=2$，包裹为 $1, 1, 1, 2, 2, 4$。
    - **从小合并**：$(1+1)=2 \to (1+2)=3 \to (2+2)=4 \to (3+4)=7$，最终算出来最大袋子是 **7**。
    - **最优方案**：$(4, 1, 1)=6$ 和 $(2, 2, 1)=5$，最大袋子是 **6**。

## 二、 核心思维 Trick：高位往低位拆分

### 1. 为什么必须“从高位往低位”处理？

由于包裹重量均为 $2^i$：

- **向上合并难**：低位碎片拼不成固定的大空间（由于袋子上限限制，不同袋子间不能跨袋组合）。
- **向下拆分完美**：一个 $2^k$ 的空闲空间，拆分成两个 $2^{k-1}$ 的空间**绝无任何浪费**。

因此，我们判定“$N$ 个容量上限为 $X$ 的袋子能否装下所有包裹”时，必须**从高位（如 $i=60$）向下遍历到低位（$i=0$）**。

### 2. 槽位（Slot）的维护过程

对于每个重量等级 $2^i$：

  

```
[上一层剩余的 2^(i+1) 槽位] 
          │
          ▼ 裂变（x 2）
[ 继承获得的 2^i 槽位 ] + [ X 的第 i 位给 N 个袋子贡献的 2^i 槽位 ]
          │
          ▼ 汇总为当前层总可用槽位 rest
   尝试装载 A[i] 个 2^i 包裹
          │
  ┌───────┴───────┐
  ▼               ▼
rest < A[i]     rest >= A[i]
(空间不足)      (扣除 A[i] 后剩余，传给下层 i-1)
  │               │
  ▼               ▼
return false     rest -= A[i]
```

1. **继承并裂变**：上一层剩下的每一个 $2^{i+1}$ 槽位，到本层分裂成 $2$ 个 $2^i$ 槽位：
$$\text{rest} = \text{rest} \times 2$$
    
2. **容量 $X$ 新增贡献**：若 $X$ 的二进制第 $i$ 位为 $1$，则 $N$ 个袋子各提供 $1$ 个 $2^i$ 槽位：
$$\text{if } ((X \gg i) \& 1) \implies \text{rest} += N$$
    
3. **剪枝防溢出**：若 $\text{rest} > \text{包裹总数}$，强制令 $\text{rest} = \text{包裹总数}$（防止 `long long` 翻倍溢出变成负数）。

4. **消耗与校验**：
    
      
    - 若 $\text{rest} < A_i$：说明目前能装 $2^i$ 的空间已经不够装 $A_i$ 个包裹了，更小尺寸的槽位不可能装得下 $2^i$ 包裹，**判定失败，`return false`**。

    - 若 $\text{rest} \ge A_i$：扣除消耗 $\text{rest} -= A_i$，剩余的 `rest` 留给下一层 $i-1$ 裂变。

## 三、 代码模板（重点关注 `check` 函数）

C++

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

vector<ll> a;
ll n, m, total_a;

bool check(ll x) {
    if (x == 0) return total_a == 0; // 特判 x = 0 避免 clzll 产生 UB
    
    // 确定最高检查位
    ll pos = max(m - 1, 63ll - __builtin_clzll(x));
    ll rest = 0;
    
    for (ll i = pos; i >= 0; i--) {
        // 1. 上层槽位裂变 + 本层 X 贡献的槽位
        rest += rest;
        if (i < 62 && ((x >> i) & 1)) rest += n;
        
        // 2. 剪枝：防止 rest 连续翻倍导致 long long 正溢出变负数
        if (rest > total_a) rest = total_a;

        // 3. 校验并消耗 A[i]
        if (i < m && a[i]) {
            if (rest >= a[i]) rest -= a[i];
            else return false; // 槽位不足，装不下 2^i 包裹
        }
    }
    return true;
}

void solve() {
    cin >> n >> m;
    a.assign(m, 0);
    total_a = 0;
    
    ll l = 0, r = 0;
    for (int i = 0; i < m; i++) {
        cin >> a[i];
        total_a += a[i];
        if (a[i]) {
            r += a[i] * (1ll << i);
            l = max(l, 1ll << i); // 至少能放下最大的单包裹
        }
    }
    r = min(r, (ll)2e18);

    // 二分答案
    while (l < r) {
        ll mid = l + (r - l) / 2;
        if (check(mid)) r = mid;
        else l = mid + 1;
    }
    cout << l << "\n";
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t;
    if (cin >> t) while (t--) solve();
}
```

## 四、 避坑指南（易错点回顾）

1. **`rest` 溢出**：`rest` 随着 `i` 的循环不断 `*= 2`，如果不做 `rest = min(rest, total_a)` 的上限限制，很快就会超出 `long long` 变成负数，导致误判 `false`。
    
      
    
2. **多组数据清空**：`a.assign(m, 0)` 必须重置大小，防止读取到上一组测试数据的残留数据。
    
      
    
3. **`__builtin_clzll(0)` UB**：当传入 $x=0$ 时会导致 CPU 异常崩溃，需要提前特判。

---
title: 倒推计数Good Schedule
date: 2026-06-03 17:03:01
categories: [题解]
tags: [Codeforces]
---
> 🔗 **原题链接**：[Problem - D - Codeforces](https://codeforces.com/contest/2230/problem/D)
题目大意：
## 1. 题目核心转化

- **看剧规则**：所有人必须严格从第 1 集开始看（$1 \rightarrow 2 \rightarrow 3 \dots$）。
    
- **合法区间**：在区间 $[L, R]$ 内，每一天 Alice 和 Bob 要么看同一集，要么都不看。一旦出现某天一人看了某集、另一人没看，就会发生**剧透（冲突）**，区间此后失效。
    
- **关键单调性**：若 $[L, R]$ 合法，则 $[L, R-1]$ 必合法。因此，对于每个固定的左端点 $L$，我们只需要求出它能延伸到的**最大右端点 $R_{max}$**。
    
- 对于每个 $L$，其对答案的贡献为：$R_{max} - L + 1$。
    

## 2. 动态规划设计：崩溃日预言法

为了避免 $O(n^2)$ 的暴力搜索，我们采用从后往前（从 $n$ 到 $1$）的动态规划。

### 辅助数组

- `pa[x]`：在当前天数之后，Alice 下一次看到第 `x` 集的具体日子。
    
- `pb[x]`：在当前天数之后，Bob 下一次看到第 `x` 集的具体日子。
    

### DP 状态设计

> **$dp[i]$ 的含义**：若 Alice 和 Bob 在第 $i$ 天**刚好同步看完**了某一集，顺着这个势头看下去，他们未来注定会在**哪一天发生剧透冲突（崩盘）**。

_注：$dp[i]$ 存的是一个绝对的“日子（天数）”，而不是长度。若能看到最后而不冲突，则预言崩溃日为 $n + 1$。_


### 状态转移方程

遍历到第 $L$ 天时，先更新 `pa` 和 `pb`。若发现 $a[L] == b[L]$（假设都是第 $v$ 集），说明今天同步成功。他们接下来想看下一集 $nxt = v + 1$：

|**下一集 nxt 的出现情况**|**命运走向（状态转移）**|**原因说明**|
|---|---|---|
|**情况一：`pa[nxt] != pb[nxt]`**|$dp[L] = \min(pa[nxt], pb[nxt])$|两人在不同日子看下一集，先到的那个人会触发剧透，该天即为崩溃日。|
|**情况二：`pa[nxt] == pb[nxt]`**|$dp[L] = dp[pa[nxt]]$|两人能在同一天安全看完下一集，接下来的命运直接绑定（继承）那一天看完后的结果。|
|**情况三：`nxt > n`**|$dp[L] = n + 1$|后面已经没有下一集了，永远不会再有新冲突。|

## 3. 算法完整执行流程

1. **倒序循环**：令 $L$ 从 $n$ 递减到 $1$。
    
2. **维护辅助表**：更新 `pa[a[L]] = L`，`pb[b[L]] = L`。
    
3. **更新当前天的 DP 值**：若 $a[L] == b[L]$，根据上述表格的转移方程计算 $dp[L]$。
    
4. **计算以 $L$ 为起点的答案**：
    
    - 因为看剧必须从第 1 集开始，查看第 1 集各自第一次出现的日子 `pa[1]` 和 `pb[1]`。
        
    - 若 `pa[1] != pb[1]`，则崩溃日为 $\min(pa[1], pb[1])$。
        
    - `pa[1] == pb[1]`，则说明第一集同步看完，崩溃日为 $dp[pa[1]]$。
        
    - **最大安全右端点** $R_{max} = \text{崩溃日} - 1$。
        
    - 若 $R_{max} \ge L$，累加贡献值 $R_{max} - L + 1$。
        

## 4. 规范代码实现

C++

```
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void solve() {
    int n;
    if (!(cin >> n)) return;

    vector<int> a(n + 1);
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
    }

    vector<int> b(n + 1);
    for (int i = 1; i <= n; ++i) {
        cin >> b[i];
    }

    vector<int> pa(n + 2, n + 1);
    vector<int> pb(n + 2, n + 1);
    vector<int> dp(n + 1, n + 1);

    long long total_segments = 0;

    for (int L = n; L >= 1; --L) {
        pa[a[L]] = L;
        pb[b[L]] = L;

        if (a[L] == b[L]) {
            int v = a[L];
            int nxt = v + 1;
            if (nxt > n) {
                dp[L] = n + 1;
            } else {
                if (pa[nxt] == pb[nxt]) {
                    dp[L] = dp[pa[nxt]];
                } else {
                    dp[L] = min(pa[nxt], pb[nxt]);
                }
            }
        }

        int R_max;
        if (pa[1] == pb[1]) {
            R_max = dp[pa[1]] - 1;
        } else {
            R_max = min(pa[1], pb[1]) - 1;
        }

        if (R_max >= L) {
            total_segments += (R_max - L + 1);
        }
    }

    cout << total_segments << "\n";
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int multTestQ;
    if (cin >> multTestQ) {
        while (multTestQ--) {
            solve();
        }
    }
    return 0;
}
```

## 5. 复杂度分析

- **时间复杂度**：$O(n)$。我们仅对天数进行了一次倒序遍历，内部所有操作（数组更新、条件判断、状态转移）均为 $O(1)$ 级别。
    
- **空间复杂度**：$O(n)$。消耗在存储输入、`pa`、`pb` 以及 `dp` 数组上，完全符合题目 512 MB 的限制。

---
title: 题解：Carrot Chopdown
date: 2026-07-20 10:17:00
categories: [题解]
tags: [Codeforces, 思维]
---
---

# 题解：Carrot Chopdown

> 这篇题解不按常规"从题意到代码"写，而是**顺着我实际踩的坑**写：先给出核心结论，然后把每一个"我当时想错/没想通"的地方单独拎出来讲透。

---
```cpp
// Problem: B2. Carrot Chopdown (Hard Version)
// Contest: Codeforces - Codeforces Round 1118 (Div. 2)
// URL: https://codeforces.com/contest/2258/problem/B2
// Memory Limit: 256 MB
// Time Limit: 1000 ms
// 
// Powered by CP Editor (https://cpeditor.org)

#include<bits/stdc++.h>
#define fr first
#define se second
#define rep(i,n) for(int i=0;i<n;i++)
#define rep1(i,n) for(int i=1;i<=n;i++)
#define pb push_back
#define all(c) c.begin(),c.end()
#define debug(x) cout<<#x<<" = "<<x<<endl;
#define IOS ios::sync_with_stdio(false),cin.tie(nullptr);
using namespace std;
typedef long long ll;
typedef pair<int,int> pr;
void solve()
{
	ll n,m,sum=0;
	cin>>n>>m;
	vector<ll> a(n+10),cnt(m+10),sf(m+10);
	rep1(i,n) cin>>a[i],sum+=a[i],cnt[a[i]]++;
	for(int i=m;i>=1;i--) sf[i] = sf[i+1] + cnt[i];
	rep1(i,1)//k
	{
		if(i>=20)
		{
			cout<<sum<<" ";
			continue;
		}
		ll tmp = (1<<i),ans=0;
		if(tmp>=m)
		{
			cout<<sum<<" ";
			continue;
		}
		for(int j=1;j<=m;j++)//x x<=m	
		{
			ll res = 0;
			//对于每个ai找满足q>=1且x*q<=ai的个数
			//这样要遍历ai实践复杂度为O(n)
			//从q的视角来看 我们枚举外层x 内层 q<=m/x 可以发现为调和级数
			//那么对于每个q 我们直接看贡献
			//发现q*x<=ai x固定了只要值大于等于qx的ai，都可以做一次贡献
			//所以，我们做一个后缀即可
			//同时由于 min(ai/x,tmp-1) + (ai==tmp*x)
			//枚举的 q 是线编号，满足 q ≤ min(tmp-1, ⌊ai/x⌋)；上界 tmp-1 来自封顶，跟 ai/x 的大小无关。ai/x 可以远大于 tmp
			//我们枚举的q就是满足q<=ai/x的这个q的个数 所以q<=ai/x<=tmp
			for(int k=1;k<=n;k++)//q
				res += min(a[k]/j,tmp-1)+(a[k]==tmp*j);
			ans = max(ans,res);
		}
		cout<<ans<<" ";
	}
	cout<<endl;
}

int main()
{
	IOS
	int _=1;
	cin>>_;
	while(_--)
		solve();
}
```
## 一、核心结论（先背下来）

设 `cnt[v]` = 长度**正好**是 v 的胡萝卜根数，`sf[v]` = 长度 **≥ v** 的根数。

对每个 `k`，记 `T = 2^k`，答案是：

$$
\text{ans}(k)=\max_{1\le x\le \lfloor m/T\rfloor}\Bigg[\underbrace{\sum_i \min\!\Big(\Big\lfloor \tfrac{a_i}{x}\Big\rfloor,\,T-1\Big)}_{\text{普通部分}}+\underbrace{cnt(Tx)}_{\text{特殊加成}}\Bigg]
$$

并且当 `T ≥ m` 时，答案直接就是 `sum = Σa_i`。

代码骨架：

```cpp
for (每个 k) {
    T = 2^k;
    if (T >= m) { 输出 sum; continue; }
    ans = 0;
    for (x = 1; x*T <= m; x++) {
        now = 0;
        for (q = 1; q < T; q++) now += sf[x*q];   // 普通部分
        now += cnt[x*T];                           // 特殊加成
        ans = max(ans, now);
    }
    输出 ans;
}
```

下面把三个"上界/写法"的坑逐个讲清楚——它们正是我出错的地方。

---

## 二、坑 1：`x` 只能枚举到 `⌊m/T⌋`

**❌ 我当时的写法：** `for (x = 1; x <= n; x++)`（或 `x <= m`）。

**为什么错：** 当 `x` 很大时，连最长的胡萝卜都切不满 `T` 段，这时成绩反而更差，多枚举纯属浪费时间（还可能是错的）。

**为什么正确上界是 `⌊m/T⌋`：**

取 `X = ⌊m/T⌋`。对任意 `x > X`：

1. **加成消失**：`x·T ≥ (X+1)·T > m`，所以不可能有 `a_i = Tx`，即 `cnt(Tx) = 0`。
2. **封顶消失**：因为 `m/(X+1) < T`，所以
   $$
   \Big\lfloor\tfrac{a_i}{x}\Big\rfloor \le \Big\lfloor\tfrac{m}{X+1}\Big\rfloor \le T-1,
   $$
   连最长那根都够不到封顶 `T-1`。于是对这类 `x`：
   $$
   S(x)=\sum_i\Big\lfloor\tfrac{a_i}{x}\Big\rfloor =: F(x)
   $$
   而 `F(x)` **随 x 增大只会变小**（分母变大，商变小）。

3. **和边界值比**：`x = X` 时，
   $$
   S(X)=\sum_i\min(\lfloor a_i/X\rfloor,T-1)+cnt(TX)\ \ge\ \sum_i\lfloor a_i/(X+1)\rfloor = F(X+1)\ \ge\ F(x).
   $$
   （关键：`X+1 > m/T` ⇒ `⌊a_i/(X+1)⌋ ≤ T-1` 没被封顶；而 `⌊a_i/X⌋ ≥ ⌊a_i/(X+1)⌋`，封顶后仍然不小于。）

**结论：** 任何 `x > ⌊m/T⌋` 都不如 `x = ⌊m/T⌋`，所以循环到 `m/T` 就够。✅

---

## 三、坑 2：内层 `q` 只能是 `1 … T-1`（即 `q < T`）

**❌ 我当时的写法：** `for (q = 1; q <= min(T, m); q++)`。

**为什么错：** 封顶是 `T-1`，只数到第 `T-1` 个倍数，写成 `q <= T` 会多算一项。

**为什么是 `q < T`：**

把 `min(⌊a/x⌋, T-1)` 理解成"**在 `x, 2x, …, (T-1)x` 里，有几个 ≤ a**"：

$$
\min\!\Big(\Big\lfloor\tfrac{a}{x}\Big\rfloor,T-1\Big)=\#\{\,q:1\le q\le T-1,\ xq\le a\,\}
$$

数的是 `q = 1, 2, …, T-1`，一共 `T-1` 个 → 正好是 `q < T`。

举例：外层 `k=2` ⇒ `T=4`，`q` 只能取 `1,2,3`。

| a | ⌊a/1⌋ | min(·, 3) | 在 {1,2,3} 中 ≤ a 的个数 |
|---|---|---|---|
| 3 | 3 | 3 | 1,2,3 → 3 |
| 4 | 4 | 3 | 1,2,3 → 3（`q=T=4` **不算**）|
| 5 | 5 | 3 | 1,2,3 → 3 |

第 `q = T` 个倍数（即 `Tx`）属于"**正好切满**"的特殊情况，交给 `cnt(Tx)` 单独算。若这里也加上，就多算了 `#{a ≥ Tx}`，比应有的 `#{a = Tx}` 大 → 答案偏大。

（顺便：`if (x*q > m) break;` 是个小优化——此时 `sf[x*q]=0`，加了也白加。）

---

## 四、坑 3：特殊项必须是 `cnt[x*T]`，不是 `cnt[T]`

**❌ 我当时的写法：** `now += cnt[T];`（把 `x` 写死了）。

**为什么错：** 特殊加成的含义是"**长度正好等于 `T·x`** 的胡萝卜根数"，它**跟着 `x` 走**。写成 `cnt[T]` 相当于只对 `x=1` 正确，其余全错。

**正确写法：** `now += cnt[x*T];`

（如果不想用 `cnt`，也可以用 `sf[x*T] - sf[x*T+1]`，两者等价。）

---

## 五、坑 4：不要凭"感觉"往公式里加项

**❌ 我当时的写法：** `res += (T-1)*(cnt[m] - cnt[min(T,m)]);`

**为什么错：** 公式里**根本没有这一项**。它是"我脑补的规则"，会导致答案偏大。

**纪律：** 代码的每一行都要能对上公式的某一部分；对不上就删掉。特别地，能证明 `S(x) ≤ sum` 恒成立（每根胡萝卜贡献 ≤ 它的长度），所以任何让答案 `> sum` 的项一定是错的。

---

## 六、不理解的思路（重点补三个）

### 思路 A：为什么要用 `sf` 累加，而不是直接算除法？

直接用 `⌊a_i/x⌋` 也能算，但复杂度是 `O(n)` 每次。用 `sf` 的好处是：

1. `sf[x*q]` 一次就给出"长度 ≥ xq 的根数"，**天然对应"打勾表的列"**；
2. 封顶 `T-1` 通过**限制 `q` 的个数**自然实现，不用 `min`；
3. 特殊项用 `cnt` 一次取到。

### 思路 B："横着数 = 竖着数"（为什么能用 `sf` 求和）

对固定的 `x`，画一张"胡萝卜 × 倍数"的打勾表，勾表示 `xq ≤ a_i`：

| | q=1 | q=2 | q=3 | 行和 |
|---|---|---|---|---|
| a=2 | ✔ | ✔ | ✘ | 2 |
| a=3 | ✔ | ✔ | ✔ | 3 |
| a=4 | ✔ | ✔ | ✔ | 3 |
| **列和** | **sf[1]=3** | **sf[2]=3** | **sf[3]=2** | |

- **横着加**（按胡萝卜）＝ `Σᵢ min(⌊aᵢ/x⌋, T-1)`；
- **竖着加**（按倍数）＝ `Σ_{q=1}^{T-1} sf[xq]`。

两种加法的**总和当然一样**（就是把同一堆勾换个顺序数）。竖着加更适合用预处理好的 `sf` 直接查。

### 思路 C：为什么 `T ≥ m` 时答案就是 `sum`

- 上界：每根胡萝卜贡献 ≤ 它自己的长度（切出来的段总长不会超过原长），所以 `S(x) ≤ Σaᵢ = sum`。
- 可达：取 `x=1`，则 `⌊aᵢ/x⌋ = aᵢ`。当 `T ≥ m` 时 `min(aᵢ, T-1) = aᵢ`（`aᵢ ≤ m ≤ T`，若 `aᵢ=T=m` 则由 `cnt(T)` 补回那 1），于是 `S(1) = sum`。

所以 `T ≥ m` 时直接输出 `sum`，后面的 `k` 全部一样，不用再算。

---

## 七、正确代码

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

int main(){
    int n, m;
    if (scanf("%d %d", &n, &m) != 2) return 0;

    vector<ll> cnt(m + 2, 0);   // cnt[v] = 长度正好 v 的根数
    ll sum = 0;
    for (int i = 0; i < n; ++i){
        int a; scanf("%d", &a);
        sum += a;
        cnt[a]++;
    }

    // sf[v] = 长度 >= v 的根数；注意从大到小递推
    vector<ll> sf(m + 3, 0);
    for (int v = m; v >= 1; --v) sf[v] = sf[v + 1] + cnt[v];

    for (int k = 1; k <= m; ++k){
        if (k >= 62) { printf("%lld ", sum); continue; }
        ll T = 1LL << k;
        if (T >= m) { printf("%lld ", sum); continue; }

        ll ans = 0;
        for (int x = 1; (ll)x * T <= m; ++x){   // 坑1：x 只到 m/T
            ll now = 0;
            for (int q = 1; q < T; ++q){        // 坑2：q 只到 T-1
                if ((ll)x * q > m) break;       // sf[x*q] 必为 0
                now += sf[x * q];
            }
            now += cnt[x * T];                  // 坑3：特殊项跟着 x
            ans = max(ans, now);
        }
        printf("%lld ", ans);
    }
    return 0;
}
```

---

## 八、样例自检

输入 `n=3, m=4, A = [2,3,4]`：

| k | T | 可用的 x | 结果 |
|---|---|---|---|
| 1 | 2 | 1, 2 | `S(1)=4, S(2)=4` → **4** |
| 2 | 4 | 1 | `S(1)=9` → **9** |
| 3 | 8 ≥ m | — | sum = **9** |
| 4 | 16 ≥ m | — | sum = **9** |

输出：`4 9 9 9`。

---

## 九、复杂度

非平凡的 `k` 只有 `O(log m)` 个（`2^k < m`），每个 `k` 的枚举量约 `O(m)`，所以

- **时间：** `O(m log m)`（加上输出 m 个答案本身的 `O(m)`）；
- **空间：** `O(m)`。

---

## 十、提交前自查清单

1. `x` 上界是不是 `m/T`？（坑 1）
2. 内层是不是 `q < T`，不含 `T`？（坑 2）
3. 特殊项是不是 `cnt[x*T]`，跟着 `x`？（坑 3）
4. 有没有自己"脑补"加进去的项？每行都能对上公式吗？（坑 4）
5. `T ≥ m` 是否直接输出 `sum`？`2^k` 是否用 `long long` 防溢出？
6. 小样例对拍：`A=[2,3,4], m=4` 应输出 `4 9 9 9`。

只要这 6 条过了，之前的那些错误就都堵住了。

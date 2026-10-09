---
title: KMP 与马拉车 · 题解
date: 2025-07-17 16:06:02
categories: [题解]
tags: []
---
A,B为上课时板子题，不过多赘述

# **A.KMP**
```cpp


A.KMP

#include<bits/stdc++.h>

using namespace std;

const int N = 1e5+10;

int ne[N];

int n,m;

string s,p;

void init() {

    ne[0] = 0;

    for(int i = 1, j = 0; i < m; i++) {

        while(j && p[i] != p[j]) j = ne[j-1];

        if(p[i] == p[j]) j++;

        ne[i] = j;

    }

}

int main() {

    cin >> m >> p >> n >> s;

    init();

    for(int i = 0, j = 0; i < n; i++) {

        while(j && s[i] != p[j]) j = ne[j-1];

        if(s[i] == p[j]) j++;

        if(j == m) {

            cout << i - m + 1 << ' ';

            j = ne[j-1];

        }

    }

    cout << endl;

    return 0;

}
```

# **B.马拉车**
```cpp
#include<bits/stdc++.h>

using namespace std;

const int N = 1.2*3e7;

int rl[N]; //臂长

int main()

{

	string a,s="#";
	
	cin>>a;
	
	int res = 0;
	
	for(int i=0;i<a.length();i++)
	
	(s+=a[i])+="#"; //插入特殊字符
	
	int len = s.size();//保存长度避免重复调用size

//cout<<s<<endl;

for(int i=1,c=0;i<len;i++)

{

	if (c + rl[c] > i) rl[i]= min(rl[2 * c - i], c + rl[c] - i);//两种情况
	
		while (i - rl[i] >= 0 && s[i - rl[i]] == s[i + rl[i]]) ++rl[i];
	
	--rl[i];

	if (i + rl[i] > c + rl[c]) c = i; //注意更新C的位置

	res = max(res, rl[i]);

}

cout<<res<<endl;

}
```
# **C.好像是思维题？**
给一个挖空的 01 串，填空后最小化串的逆序对数量。  
$n ≤ 106, ∑ n ≤ 2 × 106$。
贪心的，我们填充的数值必然是一段 1 后一段 0。如果填充  
的有 0 在 1 前面，则交换它们必然不劣。
```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 2e6+10;
void solve()
{

ll n,res=0,cnt0=0,cnt1=0,tmp;

cin>>n;

string s;

cin>>s;

for(int i=0;i<n;i++)

{

if(s[i]=='0'||s[i]=='?') cnt0++,res+=cnt1;//假定'?'为0
else cnt1++;

}
tmp = res,cnt1=0;//cnt1记录1的前缀和
for(int i=0;i<n;i++)
{

if(s[i]=='?')

	{
		
		cnt0--;
		
		tmp=tmp+cnt0-cnt1;
		
		cnt1++;
		
		res = max(res,tmp);
	
	}else if(s[i]=='0') cnt0--;//cnt0是后缀
	
	 else cnt1++;
	
	}
	
	cout<<res<<endl;

}

  

int main()

{

	ios::sync_with_stdio(false);
	
	cin.tie(nullptr);
	
	int _=1;
	
	cin>>_;
	
	while(_--)
	
	solve();
	
	return 0;

}
```
# **D.简单题**
对于每个 $a_i$，计算将其通过循环右移归零所需的最小次数，记为 $d_i$。最终答案为所有 $i$ 中最大的 $d_i$。

对于循环问题，我们可以将循环结构展开为链式结构（即令 $a_{i+n} = a_i$，$b_{i+n} = b_i$）。

考虑构造一个括号序列 $s$，按顺序拼接：

- $a_0$ 个左括号
    
- $b_0$ 个右括号
    
- $a_1$ 个左括号
    
- $b_1$ 个右括号
    
- ...
    
- $a_{2n-1}$ 个左括号
    
- $b_{2n-1}$ 个右括号
    

原问题中的操作对应于在 $s$ 中进行括号匹配的过程：

1. **操作**："将 $(a_i, b_i)$ 中较小者置零，较大者设为它们的差值"  
    $\Rightarrow$ **对应**："匹配 $(a_i, b_i)$ 对应的括号对"
    
2. **操作**："对 $a$ 进行循环右移"  
    $\Rightarrow$ **对应**："未匹配的左括号继续寻找可匹配的右括号"
    

$d_i$ 表示 $a_i$ 首次被归零所需的操作次数，这取决于与 $a_i$ 对应的最左侧左括号匹配的右括号 $b_j$ 的位置。

我们可以通过前缀和来寻找括号的匹配关系：

- 定义 $c_i = \sum\limits_{j=0}^i (a_j - b_j)$
    
- 定义 $p_i$ 为满足 $p_i > i$ 且 $c_{p_i} \leq c_i$ 的最小索引，则 $d_i = p_i - i$
    
- 可以通过单调栈计算所有 $p_i$
    

需要说明的是，括号匹配只是帮助理解解法的一种直观方式。当然，您也可以通过直接观察得出上述结论。

时间复杂度为 $O(\sum n)$。

```cpp
#include<bits/stdc++.h>

using namespace std;

typedef long long ll;

ll t,n,k,a[400009],stk[400009];

inline ll read(){

    ll s = 0,w = 1;

    char ch = getchar();

    while (ch > '9' || ch < '0'){ if (ch == '-') w = -1; ch = getchar();}

    while (ch <= '9' && ch >= '0') s = (s << 1) + (s << 3) + (ch ^ 48),ch = getchar();

    return s * w;

}

int main(){

    t = read();

    while (t --){

        n = read(),k = read();

        for (ll i = 1;i <= n;i += 1) a[i] = read();

        for (ll i = 1;i <= n;i += 1) a[i] = a[i + n] = a[i] - read();

        for (ll i = 1;i <= 2 * n;i += 1) a[i] += a[i - 1];

        ll tp = 0,ans = 0;

        for (ll i = 2 * n;i >= 1;i -= 1){

            while (tp && a[stk[tp]] > a[i]) tp -= 1;

            if (i <= n) ans = max(ans,stk[tp] - i);

            stk[++ tp] = i;

        }

        printf("%lld\n",ans);

    }

    return 0;

}
```
# **E.简单题**
简单的博弈论思考即可
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

int main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    cout.tie(0);

    string s;
    cin >> s;
    int rsum = 0;
    int n = s.size();
    
    for (int i = 0; i < n; i++) {
        if (s[i] == 'H') {
            rsum += n - i;
        }
    }

    if (rsum % 2 == 0) {
        cout << "Bob" << endl;
    } else {
        cout << "Alice" << endl;
    }
    
    return 0;
}
```
# **F.简单题**
给定三维空间内的若干条线段，限制其端点在一给定长方体上，求对于任  
意与坐标轴垂直的平面最多能和多少条线段相交。  
由于本题题面较长，提供了形式化题意。
考虑垂直于 x 轴切一刀的情况：对于一条线段从 x1 到 x2，它能被 x = c  
切断当且仅当 x1 ≤ c ≤ x2。  
所以问题转化为，给定 n 条线段，求最多有多少条线段覆盖同一位置。  
那么我们将线段离散化，考虑差分，对于 x1 到 x2，把 x1 加上 1，x2 + 1  
减去 1，然后求一遍前缀和即可。  
y, z 轴同理。
```cpp
#include <bits/stdc++.h>
using namespace std;

int solve(vector<pair<int, char>>& intervals) {
    sort(intervals.begin(), intervals.end());
    int max_depth = 0, current_depth = 0;
    
    for (auto& [position, type] : intervals) {
        if (type == 'L') {
            current_depth += 1;
            max_depth = max(max_depth, current_depth);
        } else {
            current_depth -= 1;
        }
    }
    
    return max_depth;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, a, b, c;
    cin >> n >> a >> b >> c;
    
    vector<pair<int, char>> x_intervals(n * 2);
    vector<pair<int, char>> y_intervals(n * 2);
    vector<pair<int, char>> z_intervals(n * 2);

    for (int i = 0; i < n; i++) {
        int x1, y1, z1, x2, y2, z2;
        cin >> x1 >> y1 >> z1 >> x2 >> y2 >> z2;
        
        x_intervals[i * 2] = {min(x1, x2), 'L'};
        x_intervals[i * 2 + 1] = {max(x1, x2), 'R'};
        
        y_intervals[i * 2] = {min(y1, y2), 'L'};
        y_intervals[i * 2 + 1] = {max(y1, y2), 'R'};
        
        z_intervals[i * 2] = {min(z1, z2), 'L'};
        z_intervals[i * 2 + 1] = {max(z1, z2), 'R'};
    }

    int result = 0;
    result = max(result, solve(x_intervals));
    result = max(result, solve(y_intervals));
    result = max(result, solve(z_intervals));
    
    cout << result;
    return 0;
}
```
## H.吃西瓜了
普通模拟即可
```cpp
#include<iostream>
#include<vector>
using namespace std;
const int N = 1e5+10;
string name[N];
int op[N];
int main(){
    int n,m,cur=0;
    cin>>n>>m;
    for(int i=0;i<n;i++) cin>>op[i]>>name[i];
    while(m--)
    {
        int a,b;
        cin>>a>>b;
        if(a==op[cur]) cur=(cur-b+n)%n;
        else cur=(cur+b)%n;
    }
    cout<<name[cur]<<endl;
    return 0;
}
```

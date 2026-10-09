---
title: abc_199d 暴力枚举+dfs序妙用
date: 2026-05-10 23:57:25
categories: [题解]
tags: [AtCoder]
---

> 🎯 **一句话题意**：N≤20 无向图三染色方案计数，按 DFS 序枚举剪枝避免 3^N。

> 🔗 **原题链接**：[D - RGB Coloring 2](https://atcoder.jp/contests/abc199/tasks/abc199_d?lang=en)

题目要求我们将一个图染成三个颜色共有多少个方案
N<=20
我们发现直接暴力 $3^n$ 时间会过大
可以发现 每个联通图的时间复杂度为 $3*2^{n-1}$ 其中n为这个连通图的点个数 我们发现总方案数等于各个连通块的方案数的积
问题到了如何写出这个暴力的搜索
1.DFS序，我们对于每个点，如果这个点没被遍历过（即之前没搜索过这个连通块）我们进入该点跑一个DFS序，依次放进一个vector内，然后我们按顺序枚举该点的颜色，同时枚举前，我们通过该点的出边去掉不能染色的颜色，此时因为是按DFS序，所以染色的顺序一定是没有问题所以不会出现漏算多算的情况
代码如下：
```cpp
void getorder(int s)
{
	order.push_back(s);
	vis[s] = 1;
	for(auto i:e[s])
		if(!vis[i]) getorder(i);
}
int dfs(int idx)
{
	if(idx==order.size()) return 1;
	int res = 0,s=order[idx];
	bool mv[] = {0,1,1,1};
	for(auto i:e[s])
		if(color[i]) mv[color[i]] = 0;
	for(int i=1;i<=3;i++)
	{
		if(!mv[i]) continue;
		color[s] = i;
		res += dfs(idx+1);
		color[s] = 0;
	}
	return res;
}
void solve()
{
	int n,m;
	cin>>n>>m;
	for(int i=1;i<=m;i++)
	{
		int x,y;cin>>x>>y;
		e[x].push_back(y),e[y].push_back(x);
	}
	ll ans = 1;
	for(int i=1;i<=n;i++)
	{
		if(!vis[i])
		{
			order.clear();
			getorder(i);
			ans*=dfs(0);
		}
	}
	cout<<ans<<endl;
}
```

2.

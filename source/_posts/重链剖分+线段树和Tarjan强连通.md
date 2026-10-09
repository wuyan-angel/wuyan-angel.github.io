---
title: 重链剖分+线段树和Tarjan强连通
date: 2025-11-27 16:57:31
categories: [笔记]
tags: [图论, 树, 线段树]
---
树链剖分用于将树分割成若干条链的形式，以维护树上路径的信息。
具体来说，将整棵树剖分为若干条链，使它组合成线性结构，然后用其他的数据结构维护信息。
**树链剖分**（树剖/链剖）有多种形式，如 **重链剖分**，**长链剖分** 和用于 Link/cut Tree 的剖分（有时被称作「实链剖分」）。大多数情况下（没有特别说明时），「树链剖分」都指「重链剖分」。
重链剖分可以将树上的任意一条路径划分成不超过 $𝑂(log⁡𝑛)$条连续的链，每条链上的点深度互不相同（即是自底向上的一条链，链上所有点的 $LCA$ 为链的一个端点）。

首先我们来看重链剖分这一部分，这里引入几个需要用到的概念：
1.重儿子：儿子子树节点最多的那个儿子节点，其余均为轻儿子。
2.轻重边：连接x及其重儿子的边叫做重边，同理连接x及其轻儿子的边叫做轻边。
3.轻重链：多条重边形成的链叫做重链，且规定每个节点都属于某一条重链。
![](/images/vault/Pasted-image-20251127170550.png)
接下来是代码实现：
$dfs1$ 用来完成$fa,son,sz,dep$ 四个数组的参数
$dfs2$ 用来完成$top,dfn,id$ 
我们先给出一些定义：
- $fa⁡(𝑥)$表示结点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 在树上的父亲。
- $dep⁡(𝑥)$ 表示结点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 在树上的深度。
- $sz⁡(𝑥)$ 表示结点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 的子树的结点个数。
- $son⁡(𝑥)$ 表示结点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 的 **重儿子**。
- $top⁡(𝑥)$ 表示结点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 所在 **重链** 的顶部结点（深度最小）。
- $dfn⁡(𝑥)$![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "\operatorname{dfn}(x)") 表示结点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 的 **DFS 序**，也是其在线段树中的编号。
- $id(𝑥)$ 表示 DFS 序所对应的结点编号，有 $id⁡(dfn⁡(𝑥)) =𝑥$。
接下来可以用数剖解决简单的问题LCA了代码如下：
```cpp
const int N = 5e5+10;
vector<int> e[N];
int n,m,root;
int f[N],top[N],sz[N],son[N],dep[N];
void dfs1(int u,int fa)
{
	f[u] = fa,sz[u] = 1,dep[u] = dep[fa]+1;
	for(auto j:e[u])
	{
		if(j!=fa)
		{
			dfs1(j,u);
			sz[u] += sz[j];
			if(sz[son[u]]<sz[j]) son[u] = j;
		}
	}
}
void dfs2(int u,int fa)
{
	top[u]=fa;
	if(!son[u]) return;
	dfs2(son[u],fa);
	for(auto j:e[u])
	{
		if(j==f[u]||j==son[u]) continue;
		dfs2(j,j);//搜轻的儿子 轻儿子一定为链头
	}
}
int lca(int x,int y)
{
	while(top[x]!=top[y])
	{
		if(dep[top[x]]<dep[top[y]]) swap(x,y);
		x = f[top[x]];
	}
	return dep[x]<dep[y]?x:y;
}
```
![](/images/vault/Pasted-image-20251127170550.png)
但是数剖的精髓不在于解决$LCA$，而是树上路径问题的解决，继续看上面的图，我们可以发现$DFN$序是连续的，我们利用这一性质，将按照$dfn$序的顺序建造线段树，
然后把树上的路径查询和修改为从当前点到$lca$上跳的过程中，转换为跳的过程中线段树对应的$dfn$序上这段路径的查询
这里使用区间查询和区间修改的线段树即可，其次对于有根树来说，如果我们要修改以当前节点为根的树的所有节点，我们通过性质可以发现这个子树的所有节点的区间为 $id[i] - id[i]+sz[i]-1$ 

对于修改u到v的最短路径上节点的值，代码如下：
```cpp
void update_path(int u,int v,int k)
{
	k%=p;
	while(top[u]!=top[v])
	{
		if(dep[top[u]]<dep[top[v]]) swap(u,v);
		update(1,id[top[u]],id[u],k);
		u = fa[top[u]];
	}
	if(dep[u]<dep[v]) swap(u,v);
	update(1,id[v],id[u],k);
}
```
类似的查询如下：
```cpp
ll query_path(int u,int v)
{
	ll res = 0;
	while(top[u]!=top[v])
	{
		if(dep[top[u]]<dep[top[v]]) swap(u,v);
		res+=query(1,id[top[u]],id[u]);
		res%=p;
		u = fa[top[u]];
	}
	if(dep[u]<dep[v]) swap(u,v);
	res = (res + query(1,id[v],id[u]))%p;
	return res;
}
```
对于修改和查询某个点为根的：
```cpp
void update_tree(int u,int k)
{
	update(1,id[u],id[u]+sz[u]-1,k);
}

ll query_tree(int u)
{
	return query(1,id[u],id[u]+sz[u]-1);
}
```

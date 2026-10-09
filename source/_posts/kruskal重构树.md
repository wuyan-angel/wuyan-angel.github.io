---
title: kruskal重构树
date: 2026-07-20 11:45:00
categories: [笔记]
tags: [树]
---
![](/images/vault/Pasted-image-20260719111940.png)
### 定义[](https://oi-wiki.org/graph/mst/#%E5%AE%9A%E4%B9%89_5 "Permanent link")

在跑 Kruskal 的过程中我们会从小到大加入若干条边．现在我们仍然按照这个顺序．

首先新建 𝑛![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "n") 个集合，每个集合恰有一个节点，点权为 0![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "0")．

每一次加边会合并两个集合，我们可以新建一个点，点权为加入边的边权，同时将两个集合的根节点分别设为新建点的左儿子和右儿子．然后我们将两个集合和新建点合并成一个集合．将新建点设为根．

不难发现，在进行 𝑛 −1![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "n-1") 轮之后我们得到了一棵恰有 𝑛![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "n") 个叶子的二叉树，同时每个非叶子节点恰好有两个儿子．这棵树就叫 Kruskal 重构树．

举个例子：
![](/images/vault/Pasted-image-20261001193909.png)
这张图的 Kruskal 重构树如下：
![](/images/vault/Pasted-image-20261001193918.png)
### 性质[](https://oi-wiki.org/graph/mst/#%E6%80%A7%E8%B4%A8_2 "Permanent link")

不难发现，原图中两个点之间的所有简单路径上最大边权的最小值 = 最小生成树上两个点之间的简单路径上的最大值 = Kruskal 重构树上两点之间的 LCA 的权值．

也就是说，到点 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 的简单路径上最大边权的最小值 ≤𝑣𝑎𝑙![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "\leq val") 的所有点 𝑦![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "y") 均在 Kruskal 重构树上的某一棵子树内，且恰好为该子树的所有叶子节点．

我们在 Kruskal 重构树上找到 𝑥![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "x") 到根的路径上权值 ≤𝑣𝑎𝑙![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "\leq val") 的最浅的节点．显然这就是所有满足条件的节点所在的子树的根节点．

如果需要求原图中两个点之间的所有简单路径上最小边权的最大值，则在跑 Kruskal 的过程中按边权大到小的顺序加边．

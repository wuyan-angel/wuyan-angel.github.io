---
title: SG函数及SG定理
date: 2026-07-20 11:31:00
categories: [笔记]
tags: []
---
[公平组合游戏 - OI Wiki](https://oi-wiki.org/math/game-theory/impartial-game/)

对于算法竞赛中博弈论最常见的为ICG问题（公平组合游戏），其中最基础的为NIM博弈，SG定理指出，所有ICG游戏都等价于一个单堆的NIM游戏

**公平博弈**（impartial game）指满足如下条件的组合博弈：
- 在任意确定状态下，所有参与者可选择的行动完全相同，仅取决于当前状态，与身份无关；
- 博弈中的同一个状态不可能多次抵达，博弈以参与者无法行动为结束，且博弈一定会在有限步后以非平局结束．
我们先引入一个简单的 巴什博弈：
有一堆总数为n的物品，2名玩家轮流从中拿取物品，每次至少拿1件，至多拿m件，不能不拿，最终将物品拿完者获胜。
可以发现先手不为(m+1)的倍数则必胜，否则一定会给对面非该情况，对面又给我们该情况，最终为m+1时，必败。

Nim 游戏：
老师共有 𝑛![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "n") 堆石子，第 𝑖![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "i") 堆有 𝑎𝑖![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "a_i") 枚石子．两名玩家轮流取走任意一堆中的任意多枚石子，但不能不取．取走最后一枚石子的玩家获胜．
对于上述问题，我们发现必败态为全0，可以定义异或为0为必败态，可以证明对于异或非0，一定可以变成异或为0，反之亦然，于是只要先手异或非0，我们都可以还给对方非0.
但如果把问题改成每次最多取一定的石子，该问题就涉及到我们的SG函数了。

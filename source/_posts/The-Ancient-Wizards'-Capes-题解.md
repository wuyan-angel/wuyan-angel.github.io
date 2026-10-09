---
title: The Ancient Wizards' Capes 题解
date: 2026-07-20 16:15:00
categories: [题解]
tags: [Codeforces]
---
[题目链接](https://codeforces.com/problemset/problem/2155/C)
题意：
有n个人站成一行每个人有一个斗篷可以选择放在左侧或者右侧，放在左侧则左侧的人都看不见该人，但右侧的可以看见，反之亦然，这个斗篷怎么放都只会影响到别人看见该人，不会影响到其他的人，特别的自己能看到自己。
思路：
考虑到暴力为指数级的，首先排除。
看了大佬的解法，都是一致的。
首先我们应该提出一个问题，什么情况无解？
我们发现当$|a[i]-a[i-1]|>1$ 时，此时必定无解，为何？
因为我们发现把i和i+1两个人绑在一起，他们能看到的人数是一样的，而单独算两个人的最大差值为1，所以可以发现只要相邻两个人差值大于1即为不成立，那我们发现需要成立当且仅当$|a[i]-a[i-1]|≤1$ 此时我们发现当两个人同向时差值为1，两个人互为逆向是差值为0，也就是说只要第一个人的方向固定了，后面所有人的方向也是固定的，所以此时仅有两种情况，我们把两种情况暴力跑一遍即可。代码如下：
```cpp
bool check(vector<int>&a,vector<int>&st)

{

int r=0,l=0;

for(int i=1;i<a.size();i++) r+=a[i];

if(r+1!=st[0]) return 0;

if(!a[0]) l++;

//for(auto i:a) cout<<i<<" ";

//cout<<endl;

for(int i=1;i<a.size();i++)

{

r-=a[i];

//cout<<l<<" "<<r<<endl;

if(l+r+1!=st[i]) return 0;

l+=(!a[i]);

}

return 1;

}

void solve()

{

int n;

cin>>n;

vector<int> a(n),op1(n),op2(n);

for(auto &i:a) cin>>i;

bool f1 =1;

op1[0] = 0,op2[0] = 1;

for(int i=1;i<n;i++) {

if(abs(a[i]-a[i-1])>1) f1=0;

if(a[i]==a[i-1]) op1[i] = !op1[i-1],op2[i] = !op2[i-1];

else op1[i] = op1[i-1],op2[i]=op2[i-1];

}

if(!f1) {

cout<<"0"<<endl;

return;

}

cout<<check(op1,a)+check(op2,a)<<endl;

}
```

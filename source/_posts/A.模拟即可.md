---
title: A.模拟即可
date: 2026-07-20 12:20:00
categories: [题解]
tags: [Codeforces, ICPC]
---
# A.模拟即可
[Problem - 1553C - Codeforces](https://codeforces.com/problemset/problem/1553/C)
```cpp
int rest1[]={4,4,3,3,2,2,1,1,0,0};
int rest2[]={5,4,4,3,3,2,2,1,1,0};
//当前这场过后还可以踢几次
int tot;
void solve()
{
	string s;
	cin>>s;
	int x,y,x1,y1;
	x = y = x1 = y1 = 0;
	for(int i=0;i<10;i++)
	{
		if(i%2==0) 
		{
			if(s[i]=='1') x++;
			else if(s[i]=='?') x1++;
		}
		else 
		{
			if(s[i]=='1') y++;
			else if(s[i]=='?') y1++;
		}
		if(x+x1>y+rest2[i]||y+y1>x+rest1[i])
		{
			cout<<i+1<<endl;
			return;
		}
	}
	cout<<10<<endl;
}
```

# B.二分答案或者贪心
[E - Gluttony](https://atcoder.jp/contests/abc144/tasks/abc144_e?lang=en)
```cpp

#define rep1(i,n) for(int i=1;i<=n;i++)
const int N = 2e5+10;
ll a[N],f[N],k,n;
bool check(ll x)
{
	ll tmp = k;
	for(int i=1;i<=n;i++)
	{
		ll now = x/f[i];
		if(now>=a[i]) continue;
		if(a[i]-now>tmp) return false;
		tmp -= (a[i]-now);
	}
	return true;
}
void solve()
{
	cin>>n>>k;
	rep1(i,n) cin>>a[i];
	rep1(i,n) cin>>f[i];
	sort(a+1,a+1+n);
	sort(f+1,f+1+n,greater<ll>());
	ll l = 0,r=(ll)1e13;
	while(l<r)
	{
		ll mid = l+r>>1;
		if(check(mid)) r = mid;
		else l = mid+1;
	}
	cout<<l<<endl;
}
```

# C.模拟题意即可
[Problem - 1405B - Codeforces](https://codeforces.com/problemset/problem/1405/B)
温馨提示 不开longlong见祖宗
```cpp
typedef long long ll;
void solve()
{
	int n;
	cin>>n;
	ll pre = 0,ans=0;
	vector<ll>a(n+10);
	for(int i=1;i<=n;i++) 
	{
		cin>>a[i];
		if(a[i]<0){
			ans+=max(0ll,-pre-a[i]);
			pre = max(0ll,pre+a[i]);
		}else pre+=a[i];
	}
	cout<<max(ans,pre)<<endl;
}
```

# D.暴力枚举+一点贪心
[Problem - 1535B - Codeforces](https://codeforces.com/problemset/problem/1535/B)
但是发现$∑n$ 只有2000 直接写个$n^2$秒了
```cpp
void solve()
{
	int n;
	cin>>n;
	ll ans = 0;
	vector<int>a(n);

	for(int i=0;i<n;i++)
		 cin>>a[i];
	sort(a.begin(),a.end(),[&](int x,int y){
		return x%2<y%2;
	});
	for(int i=0;i<n;i++)
		for(int j=i+1;j<n;j++)
			if(gcd(a[i],2*a[j])>1) 
				ans++;
			
	cout<<ans<<endl;
}
```

# E.并查集模板
[E - 1 or 2](https://atcoder.jp/contests/abc126/tasks/abc126_e)
```cpp
const int  N = 1e5;
int p[N];
int find(int x)
{
	if(p[x]!=x) p[x]=find(p[x]);
	return p[x];
}
void merge(int x,int y)
{
	p[find(y)] = find(x);
}
void solve()
{
	int n,m;
	cin>>n>>m;
	for(int i=1;i<=n;i++) p[i] = i;
	for(int i=1;i<=m;i++)
	{
		int x,y,z;
		cin>>x>>y>>z;
		merge(x,y);
	}
	ll ans = 0;
	for(int i=1;i<=n;i++) if(p[i]==i) ans++;
	cout<<ans<<endl;
}
```


# G.双指针或者前缀哈希即可
[Problem - B - Codeforces](https://codeforces.com/gym/103480/problem/B)
```cpp
void solve()
{
	int n,res=0,pre=0;
	cin>>n;
	vector<int>a(n);
	map<int,int>mp;
	rep(i,n) cin>>a[i];
	mp[0] = 1;
	for(int i=0;i<n;i++)
	{
		pre+=a[i];
		//debug(pre-7777)
		res+=mp[pre-7777];
		mp[pre]++;
	}
	cout<<res<<endl;
}
```

# H.adhoc 可以利用前缀异或
[Problem - 2175B - Codeforces](https://codeforces.com/problemset/problem/2175/B)
设$b_x$为a的前x项异或 则有$f(x,y)=b_{x-1}\^b_{y}$  此时发现让$b_r = b_{l-1}$ 即可然后可以发现$a_i=b_i\^b_{i-1}$ 输出即可
```cpp
void solve()
{
	int n,l,r;
	cin>>n>>l>>r;
	vector<int>a(n+10);
	for(int	i=1;i<=n;i++)
	{
		a[i]=i;
	}
	
	a[r]=l-1;
	for(int	i=1;i<=n;i++)
		cout<<(a[i]^a[i-1])<<" \n"[i==n];
}
```

# J.自定义排序
[Problem - I - Codeforces](https://codeforces.com/gym/103480/problem/I)
太简单了直接给出代码
```cpp
void solve()

{

    int n,k;

    cin>>n;

    vector<prr>a(n);

    for(int i=0;i<n;i++)

    {

        cin>>a[i].se>>a[i].fr;

    }

    sort(a.begin(),a.end(),[&](prr x,prr y)

    {

        return x.se<y.se;

    });

    cin>>k;
	cout<<a[n-k-1].fr<<endl;
```

其他题没人做出来就懒得讲了

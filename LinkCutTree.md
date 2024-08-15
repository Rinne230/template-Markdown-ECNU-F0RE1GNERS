## LCT ##

1. LCA(`询问以root为根x, y的LCA，询问格式root, x, y`)

```cpp
#include <algorithm>
#include <bits/stdc++.h>
#include <cassert>
#include <iostream>
#include <queue>
#include <vector>
#define endl '\n'
#define pll pair<i64, i64>
#define tll tuple<i64, i64, i64>
#define all(a) a.begin() + 1, a.end()
using namespace std;
using db = long double;
using i64 = long long;
const i64 N = 1e5 + 10;
const i64 mod = 998244353;
const i64 inf32 = 1e9;
const i64 inf64 = 5e18;

int n, m;
int st[N];
struct node{									
	int son[2];
	int fa, tag;
	int sz;
}tr[N];

inline int is_root(int rt){
	return tr[tr[rt].fa].son[0] != rt && tr[tr[rt].fa].son[1] != rt;
}

inline void push_down(int rt){
	if (!rt || !tr[rt].tag) return;
	tr[tr[rt].son[0]].tag ^= 1;
	tr[tr[rt].son[1]].tag ^= 1;
	tr[rt].tag ^= 1;
	swap(tr[rt].son[0], tr[rt].son[1]);
}


inline void rotate(int rt){
	int f = tr[rt].fa, gf = tr[f].fa, k = tr[f].son[0] == rt ? 0 : 1;
	if (!is_root(f)){
		if (tr[gf].son[0] == f) tr[gf].son[0] = rt;
		else tr[gf].son[1] = rt;
	}
	tr[rt].fa = gf; tr[f].fa = rt;
	tr[tr[rt].son[!k]].fa = f;
	tr[f].son[k] = tr[rt].son[!k];
	tr[rt].son[!k] = f;
}

void splay(int rt){
	int top = 0;
	st[++top] = rt;
	for (int i = rt; !is_root(i); i = tr[i].fa){
		st[++top] = tr[i].fa;
	}
	while (top){
		push_down(st[top--]);
	}
	while (!is_root(rt)){
		int f = tr[rt].fa, gf = tr[f].fa;
		if (!is_root(f)){
			if ((tr[f].son[0] == rt) ^ (tr[gf].son[0] == f)) rotate(rt);
			else rotate(f); 
		}
		rotate(rt);
	}
}


int access(int rt){
	int tmp = 0;
	while (rt){
		splay(rt);
		tr[rt].son[1] = tmp;
		tmp = rt;
		rt = tr[rt].fa;
	}
	return tmp;
}

void makeroot(int rt){
	access(rt);
	splay(rt);
	tr[rt].tag ^= 1;
}

int findroot(int rt){
	access(rt);
	splay(rt);
	int tmp = rt;
	while (tr[tmp].son[0]) tmp = tr[tmp].son[0];	
	splay(tmp);
	return tmp;
}

void cut(int x, int y){
	makeroot(x);
	access(y);
	splay(y);
	tr[y].son[0] = tr[x].fa = 0;
}

void link(int x, int y){
	makeroot(x);
	tr[x].fa = y;
}

int lca(int rt, int x, int y){
	makeroot(rt);
	access(x);
	return access(y);
}

void solve(){
	cin >> n;
	for (int i = 1, u, v; i < n; ++i){
		cin >> u >> v;
		link(u, v);
	}
	cin >> m;
	while (m--){
		int rt, x, y;
		cin >> rt >> x >> y;
		cout << lca(rt, x, y) << endl;
	}
}

int main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

2.HNOI 弹飞绵羊(`树上两点间距离，连/断`)

```cpp
#include <algorithm>
#include <bits/stdc++.h>
#include <cassert>
#include <iostream>
#include <queue>
#include <vector>
#define endl '\n'
#define pll pair<i64, i64>
#define tll tuple<i64, i64, i64>
#define all(a) a.begin() + 1, a.end()
using namespace std;
using db = long double;
using i64 = long long;
const i64 N = 2e5 + 10;
const i64 mod = 998244353;
const i64 inf32 = 1e9;
const i64 inf64 = 5e18;

int n, m;
int st[N];
struct node{									
	int son[2];
	int fa, tag;
	i64 sz;
}tr[N];

inline int is_root(int rt){
	return tr[tr[rt].fa].son[0] != rt && tr[tr[rt].fa].son[1] != rt;
}

inline void push_down(int rt){
	if (!rt || !tr[rt].tag) return;
	tr[tr[rt].son[0]].tag ^= 1;
	tr[tr[rt].son[1]].tag ^= 1;
	tr[rt].tag ^= 1;
	swap(tr[rt].son[0], tr[rt].son[1]);
}

inline void push_up(int rt){
	tr[rt].sz = tr[tr[rt].son[0]].sz + tr[tr[rt].son[1]].sz + 1;
} 

inline void rotate(int rt){
	int f = tr[rt].fa, gf = tr[f].fa, k = tr[f].son[0] == rt ? 0 : 1;
	if (!is_root(f)){
		if (tr[gf].son[0] == f) tr[gf].son[0] = rt;
		else tr[gf].son[1] = rt;
	}
	tr[rt].fa = gf; tr[f].fa = rt;
	tr[tr[rt].son[!k]].fa = f;
	tr[f].son[k] = tr[rt].son[!k];
	tr[rt].son[!k] = f;
	push_up(f);
}

void splay(int rt){
	int top = 0;
	st[++top] = rt;
	for (int i = rt; !is_root(i); i = tr[i].fa){
		st[++top] = tr[i].fa;
	}
	while (top){
		push_down(st[top--]);
	}
	while (!is_root(rt)){
		int f = tr[rt].fa, gf = tr[f].fa;
		if (!is_root(f)){
			if ((tr[f].son[0] == rt) ^ (tr[gf].son[0] == f)) rotate(rt);
			else rotate(f); 
		}
		rotate(rt);
		push_up(rt);
	}
}


int access(int rt){
	int tmp = 0;
	while (rt){
		splay(rt);
		tr[rt].son[1] = tmp;
		push_up(rt);
		tmp = rt;
		rt = tr[rt].fa;
	}
	return tmp;
}

void makeroot(int rt){
	access(rt);
	splay(rt);
	tr[rt].tag ^= 1;
}

int findroot(int rt){
	access(rt);
	splay(rt);
	int tmp = rt;
	while (tr[tmp].son[0]) tmp = tr[tmp].son[0];	
	splay(tmp);
	return tmp;
}

void cut(int x, int y){
	makeroot(x);
	access(y);
	splay(y);
	tr[y].son[0] = tr[x].fa = 0;
}

void link(int x, int y){
	makeroot(x);
	tr[x].fa = y;
}

int lca(int rt, int x, int y){
	makeroot(rt);
	access(x);
	return access(y);
}

void solve(){
	cin >> n;
	vector<int> p(n + 10);
	for (int i = 1; i <= n; ++i){
		tr[i].sz = 1;
	}
	for (int i = 1; i <= n; ++i){
		cin >> p[i];
		link(i, min(p[i] + i, n + 1));
		p[i] = min(p[i] + i, n + 1);
	}
	cin >> m;
	while(m--){
		int op;
		cin >> op;
		if (op == 1){
			int x;
			cin >> x;
			x++;	
			makeroot(n + 1);	
			access(x), splay(x);	
			cout << tr[tr[x].son[0]].sz << endl;
		}else{
			int x, y;
			cin >> x >> y;
			x++;
			int t = min(n + 1, x + y);
			cut(x, p[x]);
			link(x, t);
			p[x] = t;
		}
	}
}

int main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

3.动态并查集

```cpp
#include <algorithm>
#include <bits/stdc++.h>
#include <cassert>
#include <queue>
#include <vector>
#define endl '\n'
#define pll pair<i64, i64>
#define tll tuple<i64, i64, i64>
#define all(a) a.begin() + 1, a.end()
using namespace std;
using db = long double;
using i64 = long long;
const i64 N = 1e4 + 10;
const i64 mod = 998244353;
const i64 inf32 = 1e9;
const i64 inf64 = 5e18;

int n, m;
int st[N];
struct node{									
	int son[2];
	int fa, tag;
}tr[N];

inline int is_root(int rt){
	return tr[tr[rt].fa].son[0] != rt && tr[tr[rt].fa].son[1] != rt;
}

inline void push_down(int rt){
	if (!rt || !tr[rt].tag) return;
	tr[tr[rt].son[0]].tag ^= 1;
	tr[tr[rt].son[1]].tag ^= 1;
	tr[rt].tag ^= 1;
	swap(tr[rt].son[0], tr[rt].son[1]);
}


inline void rotate(int rt){
	int f = tr[rt].fa, gf = tr[f].fa, k = tr[f].son[0] == rt ? 0 : 1;
	if (!is_root(f)){
		if (tr[gf].son[0] == f) tr[gf].son[0] = rt;
		else tr[gf].son[1] = rt;
	}
	tr[rt].fa = gf; tr[f].fa = rt;
	tr[tr[rt].son[!k]].fa = f;
	tr[f].son[k] = tr[rt].son[!k];
	tr[rt].son[!k] = f;
}

void splay(int rt){
	int top = 0;
	st[++top] = rt;
	for (int i = rt; !is_root(i); i = tr[i].fa){
		st[++top] = tr[i].fa;
	}
	while (top){
		push_down(st[top--]);
	}
	while (!is_root(rt)){
		int f = tr[rt].fa, gf = tr[f].fa;
		if (!is_root(f)){
			if ((tr[f].son[0] == rt) ^ (tr[gf].son[0] == f)) rotate(rt);
			else rotate(f); 
		}
		rotate(rt);
	}
}


int access(int rt){
	int tmp = 0;
	while (rt){
		splay(rt);
		tr[rt].son[1] = tmp;
		tmp = rt;
		rt = tr[rt].fa;
	}
	return tmp;
}

void makeroot(int rt){
	access(rt);
	splay(rt);
	tr[rt].tag ^= 1;
}

int findroot(int rt){
	access(rt);
	splay(rt);
	int tmp = rt;
	while (tr[tmp].son[0]) tmp = tr[tmp].son[0];	
	splay(tmp);
	return tmp;
}

void cut(int x, int y){
	makeroot(x);
	access(y);
	splay(y);
	tr[y].son[0] = tr[x].fa = 0;
}

void link(int x, int y){
	makeroot(x);
	tr[x].fa = y;
}

void solve(){
	cin >> n >> m;
	while (m--){
		int x, y;
		string s;
		cin >> s;
		cin >> x >> y;
		if (s[0] == 'C') link(x, y);
		if (s[0] == 'D') cut(x, y);
		if (s[0] == 'Q'){
			if (findroot(x) == findroot(y)){
				cout << "Yes" << endl;
			}else{
				cout << "No" << endl;
			}
		}
	}
}

int main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

4.LCT(`最小生成树`)

```cpp
#include <bits/stdc++.h>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 4e5 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

int n, m, idx, ans, cnt;
namespace LCT
{
    struct node
    {
        int ch[2], w, id, fa, sz, mx, tag;
    } t[N];
#define isroot(x) ((t[t[x].fa].ch[0] != x) && (t[t[x].fa].ch[1] != x))
    void pushr(int x)
    {
        swap(t[x].ch[0], t[x].ch[1]);
        t[x].tag ^= 1;
    }
    void push_up(int x)
    {
        t[x].id = x;
        t[x].mx = t[x].w;
        if (t[t[x].ch[0]].mx > t[x].mx)
            t[x].mx = t[t[x].ch[0]].mx, t[x].id = t[t[x].ch[0]].id;
        if (t[t[x].ch[1]].mx > t[x].mx)
            t[x].mx = t[t[x].ch[1]].mx, t[x].id = t[t[x].ch[1]].id;
    }
    void push_down(int x)
    {
        if (t[x].tag)
        {
            if (t[x].ch[0])
                pushr(t[x].ch[0]);
            if (t[x].ch[1])
                pushr(t[x].ch[1]);
            t[x].tag = 0;
        }
    }
    void push_all(int x)
    {
        if (!isroot(x))
            push_all(t[x].fa);
        push_down(x);
    }
    void rotate(int x)
    {
        int y = t[x].fa;
        int z = t[y].fa;
        int k = t[y].ch[1] == x;
        if (!isroot(y))
            t[z].ch[t[z].ch[1] == y] = x;
        t[x].fa = z;
        t[y].ch[k] = t[x].ch[k ^ 1];
        t[t[x].ch[k ^ 1]].fa = y;
        t[x].ch[k ^ 1] = y;
        t[y].fa = x;
        push_up(y);
        push_up(x);
    }
    void splay(int x)
    {
        push_all(x);
        while (!isroot(x))
        {
            int y = t[x].fa;
            int z = t[y].fa;
            if (!isroot(y))
                (t[z].ch[1] == y) ^ (t[y].ch[1] == x) ? rotate(x) : rotate(y);
            rotate(x);
        }
    }
    void access(int x)
    {
        for (int y = 0; x; y = x, x = t[x].fa)
            splay(x), t[x].ch[1] = y, push_up(x);
    }
    void make_root(int x)
    {
        access(x);
        splay(x);
        pushr(x);
    }
    int find_root(int x)
    {
        access(x);
        splay(x);
        while (t[x].ch[0])
            push_down(x), x = t[x].ch[0];
        splay(x);
        return x;
    }
    void split(int x, int y)
    {
        make_root(x);
        access(y);
        splay(y);
    }
    void link(int x, int y)
    {
        make_root(x);
        if (find_root(y) != x)
            t[x].fa = y;
    }
    int ck(int x, int y)
    {
        make_root(x);
        return find_root(y) != x;
    }
    void cut(int x, int y)
    {
        make_root(x);
        if (find_root(y) == x && t[y].fa == x && !t[y].ch[0])
        {
            t[x].ch[1] = t[y].fa = 0;
            push_up(x);
        }
    }
}

void solve(){
    using namespace LCT;
    cin >> n >> m;
    idx = n;
    for (int i = 1, x, y, z; i <= m; i++)
    {
        cin >> x >> y >> z;
        t[++idx].w = z;
        if (x != y && ck(x, y))
            link(x, idx), link(idx, y), ans += z, ++cnt;
        else
        {
            split(x, y);
            int now = t[y].id;
            if (t[now].mx <= z)
                continue;
            ans -= t[now].mx - z;
            splay(now);
            t[t[now].ch[0]].fa = t[t[now].ch[1]].fa = 0;
            link(x, idx);
            link(idx, y);
        }
    }
    if (cnt < n - 1) cout << "orz" << endl;
    else cout << ans << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

5.最小生成树的运用(`树上构建最小生成树，每个边有权值a，b满足1到n的max(a) + max(b)最小`)

```cpp
#include <bits/stdc++.h>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e5 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

int n, m, ans = inf32;
namespace LCT
{
	struct Edge{
		int u, v, a, b;
	}e[N];
    int cmp(Edge a, Edge b){return a.a == b.a ? a.b < b.b : a.a < b.a;}
    struct node
    {
        int ch[2], w, id, fa, sz, mx, tag;
    } t[N];
#define isroot(x) ((t[t[x].fa].ch[0] != x) && (t[t[x].fa].ch[1] != x))
    void pushr(int x)
    {
        swap(t[x].ch[0], t[x].ch[1]);
        t[x].tag ^= 1;
    }
    void push_up(int x)
    {
        t[x].id = x;
        t[x].mx = t[x].w;
        if (t[t[x].ch[0]].mx > t[x].mx)
            t[x].mx = t[t[x].ch[0]].mx, t[x].id = t[t[x].ch[0]].id;
        if (t[t[x].ch[1]].mx > t[x].mx)
            t[x].mx = t[t[x].ch[1]].mx, t[x].id = t[t[x].ch[1]].id;
    }
    void push_down(int x)
    {
        if (t[x].tag)
        {
            if (t[x].ch[0])
                pushr(t[x].ch[0]);
            if (t[x].ch[1])
                pushr(t[x].ch[1]);
            t[x].tag = 0;
        }
    }
    void push_all(int x)
    {
        if (!isroot(x))
            push_all(t[x].fa);
        push_down(x);
    }
    void rotate(int x)
    {
        int y = t[x].fa;
        int z = t[y].fa;
        int k = t[y].ch[1] == x;
        if (!isroot(y))
            t[z].ch[t[z].ch[1] == y] = x;
        t[x].fa = z;
        t[y].ch[k] = t[x].ch[k ^ 1];
        t[t[x].ch[k ^ 1]].fa = y;
        t[x].ch[k ^ 1] = y;
        t[y].fa = x;
        push_up(y);
        push_up(x);
    }
    void splay(int x)
    {
        push_all(x);
        while (!isroot(x))
        {
            int y = t[x].fa;
            int z = t[y].fa;
            if (!isroot(y))
                (t[z].ch[1] == y) ^ (t[y].ch[1] == x) ? rotate(x) : rotate(y);
            rotate(x);
        }
    }
    void access(int x)
    {
        for (int y = 0; x; y = x, x = t[x].fa)
            splay(x), t[x].ch[1] = y, push_up(x);
    }
    void make_root(int x)
    {
        access(x);
        splay(x);
        pushr(x);
    }
    int find_root(int x)
    {
        access(x);
        splay(x);
        while (t[x].ch[0])
            push_down(x), x = t[x].ch[0];
        splay(x);
        return x;
    }
    void split(int x, int y)
    {
        make_root(x);
        access(y);
        splay(y);
    }
    void link(int x, int y)
    {
        make_root(x);
        if (find_root(y) != x)
            t[x].fa = y;
    }
    int ck(int x, int y)
    {
        make_root(x);
        return find_root(y) != x;
    }
    void cut(int x, int y)
    {
        make_root(x);
        if (find_root(y) == x && t[y].fa == x && !t[y].ch[0])
        {
            t[x].ch[1] = t[y].fa = 0;
            push_up(x);
        }
    }
}

void solve(){
	using namespace LCT;
	cin >> n >> m;
	for (int i = 1; i <= m; ++i){
		auto &[u, v, a, b] = e[i];
		cin >> u >> v >> a >> b;
	}
	sort(e + 1, e + 1 + m, cmp);
	for (int i = 1; i <= m; ++i){
		int idx = n + i;
		auto [x, y, a, b] = e[i];
		t[idx].w = b;
		if (x == y) continue;
		if (ck(x, y)){
			link(x, idx);
			link(idx, y);
		}else{
			split(x, y);
			int now = t[y].id;
			if (t[now].mx <= b) continue;
			splay(now);
			t[t[now].ch[0]].fa = t[t[now].ch[1]].fa = 0;
			link(x, idx);
			link(idx, y);
		}
		if (!ck(1, n)){
			split(1, n);
			ans = min(ans, t[n].mx + a);
		}
	}
	if (ans == inf32) cout << -1 << endl;
	else cout << ans << endl;
}

signed main(){
	ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
	int t = 1;
	//cin >> t;
	while(t--) solve();
	return 0;
}
```

6.LCT树上路径信息维护

一棵n个点的树，每个点的初始权值为1。对于这棵树有q个操作，每个操作为以下四种操作之一：

\+ u v c：将u到v的路径上的点的权值都加上自然数c；

\- u1 v1 u2 v2：将树中原有的边(u1,v1)删除，加入一条新边(u2,v2)，保证操作完之后仍然是一棵树；

\* u v c：将u到v的路径上的点的权值都乘上自然数c；

/ u v：询问u到v的路径上的点的权值和，求出答案对于51061的余数。

数据范围：1 <= n，q <= 10^5，0 <= c <= 10^4。

```cpp
#include <bits/stdc++.h>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e5 + 10;
const ll mod = 51061;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

int n, q;
namespace LCT
{
    struct node
    {
        int ch[2], id, fa, sz, tag;
        ll taga, tagc;//加法，乘法tag
        ll sum, val;
    } t[N];
#define isroot(x) ((t[t[x].fa].ch[0] != x) && (t[t[x].fa].ch[1] != x))
    void push_up(int x)
    {
        t[x].sum = (t[x].val + t[t[x].ch[0]].sum % mod + t[t[x].ch[1]].sum) % mod;
        t[x].sz = 1 + t[t[x].ch[0]].sz + t[t[x].ch[1]].sz;
    }
    void pushr(int x) 
    {
        swap(t[x].ch[0], t[x].ch[1]);
        t[x].tag ^= 1;
    }
    void pusha(int x, int c){
        t[x].taga = t[x].taga + c % mod;
        t[x].val = t[x].val + c % mod;
        t[x].sum = t[x].sum + (c * t[x].sz % mod) % mod;
    }
    void pushc(int x, int c){
        t[x].taga = t[x].taga * c % mod;
        t[x].val = t[x].val * c % mod;
        t[x].sum = t[x].sum * c % mod;
        t[x].tagc = t[x].tagc * c % mod;
    }
    void push_down(int x)
    {
        if (t[x].tag)
        {
            if (t[x].ch[0])
                pushr(t[x].ch[0]);
            if (t[x].ch[1])
                pushr(t[x].ch[1]);
            t[x].tag = 0;
        }
        if (t[x].tagc != 1){
            pushc(t[x].ch[0], t[x].tagc);
            pushc(t[x].ch[1], t[x].tagc);
            t[x].tagc = 1;
        }
        if (t[x].taga){
            pusha(t[x].ch[0], t[x].taga);
            pusha(t[x].ch[1], t[x].taga);
            t[x].taga = 0;
        }
    }
    void push_all(int x)
    {
        if (!isroot(x))
            push_all(t[x].fa);
        push_down(x);
    }
    void rotate(int x)
    {
        int y = t[x].fa;
        int z = t[y].fa;
        int k = t[y].ch[1] == x;
        if (!isroot(y))
            t[z].ch[t[z].ch[1] == y] = x;
        t[x].fa = z;
        t[y].ch[k] = t[x].ch[k ^ 1];
        t[t[x].ch[k ^ 1]].fa = y;
        t[x].ch[k ^ 1] = y;
        t[y].fa = x;
        push_up(y);
        push_up(x);
    }
    void splay(int x)
    {
        push_all(x);
        while (!isroot(x))
        {
            int y = t[x].fa;
            int z = t[y].fa;
            if (!isroot(y))
                (t[z].ch[1] == y) ^ (t[y].ch[1] == x) ? rotate(x) : rotate(y);
            rotate(x);
        }
    }
    void access(int x)
    {
        for (int y = 0; x; y = x, x = t[x].fa)
            splay(x), t[x].ch[1] = y, push_up(x);
    }
    void make_root(int x)
    {
        access(x);
        splay(x);
        pushr(x);
    }
    int find_root(int x)
    {
        access(x);
        splay(x);
        while (t[x].ch[0])
            push_down(x), x = t[x].ch[0];
        splay(x);
        return x;
    }
    void split(int x, int y)
    {
        make_root(x);
        access(y);
        splay(y);
    }
    void link(int x, int y)
    {
        make_root(x);
        if (find_root(y) != x)
            t[x].fa = y;
    }
    int ck(int x, int y)
    {
        make_root(x);
        return find_root(y) != x;
    }
    void cut(int x, int y)
    {
        make_root(x);
        if (find_root(y) == x && t[y].fa == x && !t[y].ch[0])
        {
            t[x].ch[1] = t[y].fa = 0;
            push_up(x);
        }
    }
}

void solve(){
	using namespace LCT;
	cin >> n >> q;
    for (int i = 1; i <= n; ++i) {
        t[i].sz = t[i].tagc = 1;
        t[i].val = t[i].sum = 1;
        t[i].taga = 0;
    }
    for (int i = 1; i < n; ++i){
        int u, v;
        cin >> u >> v;
        link(u, v);
    }
    while (q--){
        char op;
        int x, y, z;
        cin >> op;
        if (op == '+'){
            cin >> x >> y >> z;
            split(x, y);
            pusha(y, z);
        }else if (op == '-'){
            cin >> x >> y;
            cut(x, y);
            cin >> x >> y;
            link(x, y);
        }else if (op == '*'){
            cin >> x >> y >> z;
            split(x, y);
            pushc(y, z);
        }else{
            cin >> x >> y;
            split(x, y);
            cout << t[y].sum % mod << endl;
        }
    }
}

signed main(){
	ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
	int t = 1;
	//cin >> t;
	while(t--) solve();
	return 0;
}
```

7.大融合(`子树大小查询`)

给定n个点，进行Q次操作，操作有两种：

A x y ：在x,y之间连一条边，保证x,y不连通

Q x y ：求通过边(x,y)的简单路径数量，保证存在边(x,y)

N,Q <= 10^5

```cpp
#include <bits/stdc++.h>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e5 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

namespace LCT
{
    struct node
    {
        int ch[2], fa, tag;
        int sz, szx; //sz实子树，szx虚子树    
    } t[N];
#define isroot(x) ((t[t[x].fa].ch[0] != x) && (t[t[x].fa].ch[1] != x))
    void pushr(int x)
    {
        swap(t[x].ch[0], t[x].ch[1]);
        t[x].tag ^= 1;
    }
    void push_up(int x)
    {
        t[x].sz = 1 + t[t[x].ch[0]].sz + t[t[x].ch[0]].szx + t[t[x].ch[1]].sz + t[t[x].ch[1]].szx;
    }
    void push_down(int x)
    {
        if (t[x].tag)
        {
            if (t[x].ch[0])
                pushr(t[x].ch[0]);
            if (t[x].ch[1])
                pushr(t[x].ch[1]);
            t[x].tag = 0;
        }
    }
    void push_all(int x)
    {
        if (!isroot(x))
            push_all(t[x].fa);
        push_down(x);
    }
    void rotate(int x)
    {
        int y = t[x].fa;
        int z = t[y].fa;
        int k = t[y].ch[1] == x;
        if (!isroot(y))
            t[z].ch[t[z].ch[1] == y] = x;
        t[x].fa = z;
        t[y].ch[k] = t[x].ch[k ^ 1];
        t[t[x].ch[k ^ 1]].fa = y;
        t[x].ch[k ^ 1] = y;
        t[y].fa = x;
        push_up(y);
        push_up(x);
    }
    void splay(int x)
    {
        push_all(x);
        while (!isroot(x))
        {
            int y = t[x].fa;
            int z = t[y].fa;
            if (!isroot(y))
                (t[z].ch[1] == y) ^ (t[y].ch[1] == x) ? rotate(x) : rotate(y);
            rotate(x);
        }
    }
    void access(int x)
    {
        for (int y = 0; x; y = x, x = t[x].fa){
            splay(x);
            if (t[x].ch[1])
                t[x].szx += t[t[x].ch[1]].sz + t[t[x].ch[1]].szx;
            t[x].ch[1] = y;
            if (y)
                t[x].szx -= t[y].sz + t[y].szx;
            push_up(x);
        }
            
    }
    void make_root(int x)
    {
        access(x);
        splay(x);
        pushr(x);
    }
    int find_root(int x)
    {
        access(x);
        splay(x);
        while (t[x].ch[0])
            push_down(x), x = t[x].ch[0];
        splay(x);
        return x;
    }
    void split(int x, int y)
    {
        make_root(x);
        access(y);
        splay(y);
    }
    void link(int x, int y)
    {
        split(x, y);
        t[x].fa = y;
        t[y].szx += t[x].sz + t[x].szx;
    }
    int ck(int x, int y)
    {
        make_root(x);
        return find_root(y) != x;
    }
    void cut(int x, int y)
    {
        make_root(x);
        if (find_root(y) == x && t[y].fa == x && !t[y].ch[0])
        {
            t[x].ch[1] = t[y].fa = 0;
            push_up(x);
        }
    }
}

void solve(){
    using namespace LCT;
    int n, q, x, y;
    cin >> n >> q;
    for (int i = 1; i <= n; ++i) t[i].sz = 1;
    while (q--){
        char op;
        cin >> op;
        if (op == 'A'){
            cin >> x >> y;
            link(x, y);
        }else{
            cin >> x >> y;
            split(x, y);
            cout << 1ll * (t[x].sz + t[x].szx) * (t[y].sz + t[y].szx - t[x].sz - t[x].szx) << endl;
        }
    }
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

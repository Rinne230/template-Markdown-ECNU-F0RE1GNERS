## 分块 ##

1.分块I(区间修改, 区间查询)

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

int n, m, sq;
int v[N], bl[N], tag[N], ans[N];

void update(int a, int b){
    for (int i = a; i <= min(bl[a] * sq, b); ++i) {
        ans[bl[a]] -= (v[i] ^ tag[bl[a]]);
        v[i] ^= 1;
        ans[bl[a]] += (v[i] ^ tag[bl[a]]);
    }
    if (bl[a] != bl[b]){
        for (int i = (bl[b] - 1) * sq + 1; i <= b; ++i) {
            ans[bl[b]] -= (v[i] ^ tag[bl[b]]);
            v[i] ^= 1;
            ans[bl[b]] += (v[i] ^ tag[bl[b]]);
        }
    }    
    for (int i = bl[a] + 1; i <= bl[b] - 1; ++i){
        tag[i] ^= 1;
        ans[i] = sq - ans[i];
    }
}

ll query(int a, int b){
    ll res = 0;
    for (int i = a; i <= min(bl[a] * sq, b); ++i) {
        res += (v[i] ^ tag[bl[a]]);
    }
    if (bl[a] != bl[b]){
        for (int i = (bl[b] - 1) * sq + 1; i <= b; ++i) {
            res += (v[i] ^ tag[bl[b]]);
        }
    }
    for (int i = bl[a] + 1; i <= bl[b] - 1; ++i) res += ans[i];
    return res;
}

void solve(){
    cin >> n >> m;
    sq = sqrt(n);
    for (int i = 1; i <= n; ++i) bl[i] = (i - 1) / sq + 1;
    for (int i = 1; i <= m; ++i){
        int op, l, r;
        cin >> op >> l >> r;
        if (op == 0) update(l, r);
        else cout << query(l, r) << endl;
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

2.分块II

给出一个长为 $n$ 的数列，以及 $n$ 个操作，操作涉及区间加法，询问区间内小于某个值 $x$ 的元素个数。

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

int n, blo;
int v[N], bl[N], atag[N];
vector<int> ve[2005];

void reset(int x)
{
    ve[x].clear();
    for (int i = (x - 1) * blo + 1; i <= min(x * blo, n); i++)
        ve[x].push_back(v[i]);
    sort(ve[x].begin(), ve[x].end());
}

void add(int a, int b, int c)
{
    for (int i = a; i <= min(bl[a] * blo, b); i++)
        v[i] += c;
    reset(bl[a]);
    if (bl[a] != bl[b])
    {
        for (int i = (bl[b] - 1) * blo + 1; i <= b; i++)
            v[i] += c;
        reset(bl[b]);
    }
    for (int i = bl[a] + 1; i <= bl[b] - 1; i++)
        atag[i] += c;
}

int query(int a, int b, int c)
{
    int ans = 0;
    for (int i = a; i <= min(bl[a] * blo, b); i++)
        if (v[i] + atag[bl[a]] < c)
            ans++;
    if (bl[a] != bl[b])
        for (int i = (bl[b] - 1) * blo + 1; i <= b; i++)
            if (v[i] + atag[bl[b]] < c)
                ans++;
    for (int i = bl[a] + 1; i <= bl[b] - 1; i++)
    {
        int x = c - atag[i];
        ans += lower_bound(ve[i].begin(), ve[i].end(), x) - ve[i].begin();
    }
    return ans;
}

int main()
{
    cin >> n;
    blo = sqrt(n);
    for (int i = 1; i <= n; i++)
        cin >> v[i];
    for (int i = 1; i <= n; i++)
    {
        bl[i] = (i - 1) / blo + 1;
        ve[bl[i]].push_back(v[i]);
    }
    for (int i = 1; i <= bl[n]; i++)
        sort(ve[i].begin(), ve[i].end());
    for (int i = 1; i <= n; i++)
    {
        int f, a, b, c;
        cin >> f >> a >> b >> c;
        if (f == 0)
            add(a, b, c);
        if (f == 1)
            printf("%d\n", query(a, b, c * c));
    }
    return 0;
}
```

3.线性RMQ

```cpp
/**
    线性RMQ
 */ 

#include <bits/stdc++.h>
#include <limits>
#define endl '\n'
using namespace std;
typedef long long ll;

const int N = 5e4 + 10;
const int M = 2e4 + 10;
const int L = 80;
const int mod = 998244353;
const int inf32 = 0x3f3f3f3f;
const ll inf64 = 4e18;

int a[N + M];
int highbit[M];
int stmax[M][L], stmin[M][L];
int premax[M][L], premin[M][L];
int sufmax[M][L], sufmin[M][L];
int quemax[M][L], quemin[M][L];
int stackmax[L], stackmin[L];

void solve(){
    int n, q;
    cin >> n >> q;
    int B = int(log2(n)); // 块的大小
    int S = (n - 1) / B + 1; // 块的个数
    for (int b = 0; b < S; ++b) stmin[b][0] = inf32;
    for (int i = 0; i < n; ++i){
        cin >> a[i];
        stmin[i / B][0] = min(stmin[i / B][0], a[i]);
        stmax[i / B][0] = max(stmax[i / B][0], a[i]);
    }
    for (int b = S - 1; b >= 0; b--){
        for (int k = 1; b + (1 << k) - 1 < S; ++k){
            stmin[b][k] = min(stmin[b][k - 1], stmin[b + (1 << (k - 1))][k - 1]);
            stmax[b][k] = max(stmax[b][k - 1], stmax[b + (1 << (k - 1))][k - 1]); 
        }
    }
    for (int b = 0; b < S; ++b){
        int be = b * B;
        premin[b][0] = premax[b][0] = a[be];
        for (int k = 1; k < B; ++k){
            premin[b][k] = min(premin[b][k - 1], a[be + k]);
            premax[b][k] = max(premax[b][k - 1], a[be + k]);
        }
        sufmin[b][B - 1] = sufmax[b][B - 1] = a[be + B - 1];
        for (int k = B - 2; k >= 0; --k){
            sufmin[b][k] = min(sufmin[b][k + 1], a[be + k]);
            sufmax[b][k] = max(sufmax[b][k + 1], a[be + k]);  
        }
    }
    for (int b = 0; b < S; ++b){
        int be = b * B;
        int spmin = 0, nowmin = 0;
        int spmax = 0, nowmax = 0;
        for (int i = 0; i < B; ++i){
            while (spmin && a[be + stackmin[spmin]] > a[be + i]) nowmin ^= 1 << stackmin[spmin--];
            while (spmax && a[be + stackmax[spmax]] < a[be + i]) nowmax ^= 1 << stackmax[spmax--];
            quemin[b][i] = (nowmin ^= 1 << (stackmin[++spmin] = i));
            quemax[b][i] = (nowmax ^= 1 << (stackmax[++spmax] = i));
        }
    }
    for (int i = 2; i <= S; ++i) highbit[i] = highbit[i >> 1] + 1;

    while(q --) {
		int l, r;
        cin >> l >> r;
        l--, r--;
		int L = l / B, R = r / B;
		int li = l % B, ri = r % B;

		int mn = inf32, mx = 0;
		if(L == R) {
			mn = min(mn, a[l + __builtin_ctz(quemin[R][ri] >> li)]);
			mx = max(mx, a[l + __builtin_ctz(quemax[R][ri] >> li)]);
		}
		else {
			mn = min(mn, sufmin[L][li]);
			mn = min(mn, premin[R][ri]);
			mx = max(mx, sufmax[L][li]);
			mx = max(mx, premax[R][ri]);
			int len = R - L - 1;
			int k = highbit[len];
			if(len) {
				mn = min(mn, stmin[L + 1][k]);
				mn = min(mn, stmin[R - (1 << k)][k]);
				mx = max(mx, stmax[L + 1][k]);
				mx = max(mx, stmax[R - (1 << k)][k]);
			}
		}
        cout << mx - mn << endl;
	}
}

signed main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```
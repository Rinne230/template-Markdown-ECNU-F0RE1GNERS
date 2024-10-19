## 分块 ##

#### 分块I(区间修改, 区间查询) ####

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

#### 分块II ####

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

#### 线性RMQ ####

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

#### 区间[l, r]有多少数字出现正偶数次 ####

```cpp
// https://www.luogu.com.cn/problem/P4135
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 1e5 + 10;
constexpr int M = 320;
constexpr int mod = 998244353;

using namespace std;

int a[N], block[N], cnt[M][N];
int L[M], f[M][M], num[N];

int n, c, q, B;

void solve(){
    cin >> n >> c >> q;
    B = sqrt(n);
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        block[i] = (i - 1) / B + 1;
        if (block[i] != block[i - 1]) L[block[i]] = i;
    }
    block[n + 1] = block[n] + 1;
    L[block[n + 1]] = n + 1;
    for (int i = 1; i <= block[n]; ++i){
        int t = 0;
        for (int j = L[i]; j <= n; ++j) {
            cnt[i][a[j]]++;
            if ((cnt[i][a[j]] & 1) && cnt[i][a[j]] > 1) t--;
            else if (cnt[i][a[j]] % 2 == 0) t++;
            if (block[j] != block[j + 1]) {
                f[i][block[j]] = t;
            }
        }
    }
    int ans = 0;
    stack<int> st;
    while(q--) {
        int l, r;
        cin >> l >> r;
        l = (l + ans) % n + 1, r = (r + ans) % n + 1;
        if (l > r) swap(l, r);
        ans = 0;
        if (block[l] == block[r]) {
            for (int i = l; i <= r; ++i) {
                num[a[i]]++;
                st.push(a[i]);
            }
            while (!st.empty()) {
                int t = st.top();
                st.pop();
                if (num[t]){
                    ans += (num[t] & 1) ^ 1;
                    num[t] = 0;
                }
            }
            cout << ans << endl;
            continue;
        }
        if (block[l] + 1 <= block[r] - 1) {
            ans = f[block[l] + 1][block[r] - 1];
        }
        for (int i = l; i < L[block[l] + 1]; ++i) {
            num[a[i]]++;
            st.push(a[i]);
        }
        for (int i = L[block[r]]; i <= r; ++i) {
            num[a[i]]++;
            st.push(a[i]);
        }
        while (!st.empty()) {
            int t = st.top();
            st.pop();
            if (!num[t]) continue;
            if (cnt[block[l] + 1][t] - cnt[block[r]][t] > 0 && (cnt[block[l] + 1][t] - cnt[block[r]][t]) % 2 == 0 && num[t] % 2) ans--;
            else if (cnt[block[l] + 1][t] - cnt[block[r]][t] > 0 && (cnt[block[l] + 1][t] - cnt[block[r]][t]) % 2 && num[t] % 2) ans++;
            else if (cnt[block[l] + 1][t] - cnt[block[r]][t] == 0 && num[t] % 2 == 0) ans++;
            num[t] = 0;
        }
        cout << ans << endl;
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

#### 区间众数查询1 ####

```cpp
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 1e5 + 10;
constexpr int M = 320;
constexpr int mod = 998244353;

using namespace std;

int n, m, q, B;
int a[N], b[N], block[N], cnt[M][N], L[M];
int c[N];
pair<int, int> f[M][M];

void solve(){
    int n, q;
    cin >> n >> q;
    B = sqrt(n);
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        b[i] = a[i];
        block[i] = (i - 1) / B + 1;
        if (block[i] != block[i - 1]) L[block[i]] = i;
    }
    block[n + 1] = block[n] + 1;
    L[block[n + 1]] = n + 1;
    sort(b + 1, b + n + 1);
    for (int i = 1; i <= n; ++i) a[i] = lower_bound(b + 1, b + n + 1, a[i]) - b;
    for (int i = 1; i <= block[n]; ++i){
        int mxcnt = 0, num = n;
        for (int j = L[i]; j <= n; ++j) {
            cnt[i][a[j]]++;
            if (cnt[i][a[j]] > mxcnt) {
                mxcnt = cnt[i][a[j]];
                num = a[j];
            }else if (cnt[i][a[j]] == mxcnt) {
                num = min(num, a[j]);
            }
            if (block[j] != block[j + 1]) {
                f[i][block[j]] = {mxcnt, num};
            }
        }
    }

    int ans = 0;
    stack<int> st;
    while (q--) {
        int l, r;
        cin >> l >> r;
        l = (l + ans - 1) % n + 1, r = (r + ans - 1) % n + 1;
        if (l > r) swap(l, r);
        ans = 0;
        int mxcnt = 0, num = n;
        if (block[l] == block[r]) {
            for (int i = l; i <= r; ++i) {
                c[a[i]]++;
                st.push(a[i]);
            }
            while (!st.empty()) {
                int t = st.top();
                st.pop();
                if (c[t]) {
                    if (c[t] > mxcnt) {
                        mxcnt = c[t];
                        num = t;
                    }else if (c[t] == mxcnt) {
                        num = min(num, t);
                    }
                    c[t] = 0;
                }
            }
            ans = b[num];
            cout << ans << endl; 
            continue;
        }
        if (block[l] + 1 <= block[r] - 1) {
            mxcnt = f[block[l] + 1][block[r] - 1].first;
            num = f[block[l] + 1][block[r] - 1].second;
        }
        for (int i = l; i < L[block[l] + 1]; ++i) {
            c[a[i]]++;
            st.push(a[i]);
        }
        for (int i = L[block[r]]; i <= r; ++i) {
            c[a[i]]++;
            st.push(a[i]);
        }
        while (!st.empty()) {
            int t = st.top();
            st.pop();
            if (!c[t]) continue;
            if (c[t] + cnt[block[l] + 1][t] - cnt[block[r]][t] > mxcnt) {
                mxcnt = c[t] + cnt[block[l] + 1][t] - cnt[block[r]][t];
                num = t;
            }else if (c[t] + cnt[block[l] + 1][t] - cnt[block[r]][t] == mxcnt) {
                num = min(num, t);
            }
            c[t] = 0;
        }
        ans = b[num];
        cout << ans << endl;
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

#### 区间众数查询2 ####

```cpp
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 5e5 + 10;
constexpr int M = 710;
constexpr int mod = 998244353;

using namespace std;

int a[N], b[N], block[N], L[M], R[M], f[M][M], tot[N], pos[N];
vector<int> v[N]; 
int B;

void solve(){
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        b[i] = a[i];
    }
    sort(b + 1, b + n + 1);
    int len = unique(b + 1, b + n + 1) - b - 1;
    for (int i = 1; i <= n; ++i) {
        a[i] = lower_bound(b + 1, b + len + 1, a[i]) - b;
    }
    for (int i = 1; i <= n; ++i) {
        v[a[i]].push_back(i);
        pos[i] = v[a[i]].size();
        pos[i]--;
    }
    int B = sqrt(n);
    for (int i = 1; i <= n; ++i) {
        block[i] = (i - 1) / B + 1;
    }
    for (int i = 1; i <= block[n]; ++i) {
        L[i] = (i - 1) * B + 1;
        R[i] = i * B;
    }
    R[block[n]] = n; 
    for (int i = 1; i <= block[n]; ++i) {
        memset(tot, 0, sizeof tot);
        for (int j = i; j <= block[n]; ++j) {
            f[i][j] = f[i][j - 1];
            for (int k = L[j]; k <= R[j]; ++k) {
                f[i][j] = max(f[i][j], ++tot[a[k]]);
            }
        }
    }

    auto query = [&](int l, int r) -> int {
        int ans = 0;
        if (block[l] == block[r]) {
            for (int i = l; i <= r; ++i) tot[a[i]] = 0;
            for (int i = l; i <= r; ++i) ans = max(ans, ++tot[a[i]]);
            return ans;
        }
        ans = f[block[l] + 1][block[r] - 1];
        for (int i = l; i <= R[block[l]]; ++i) {
            auto it = pos[i];
            while (it + ans < v[a[i]].size() && v[a[i]][it + ans] <= r) {
                ans++;
            }
        }
        for (int i = L[block[r]]; i <= r; ++i) {
            auto it = pos[i];
            while (it - ans >= 0 && v[a[i]][it - ans] >= l) ++ans;
        }
        return ans;
    };  
    int lstans = 0;
    while (m--) {
        int l, r;
        cin >> l >> r;
        l ^= lstans, r ^= lstans;
        if (l > r) swap(l, r);  
        cout << (lstans = query(l, r)) << endl;
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

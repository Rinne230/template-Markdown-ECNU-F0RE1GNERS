## Tree Dp ## 

1.树的平衡点

```cpp
function<void(int, int)> dfs = [&](int u, int fa){
    sz[u] = 1;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs(v, u);
        sz[u] += sz[v];
        int mx = max(sz[u], n - sz[u]);
        if (mx <= mn){
            mn = mx;
            id = min(u, id);
        }
    }
};
```

2.树的最小点覆盖(最少的点覆盖所有边)

```cpp
void dp(int u) {
    bool fg = 0;
    for(int i = h[u]; ~i; i = nex[i]) {
        int j = v[i];
        fg = 1;
        dp(j);
        f[u][0] += f[j][1];
        f[u][1] += min(f[j][0], f[j][1]);
    }
    f[u][1] += 1;
    if(!fg) {
        f[u][0] = 0; f[u][1] = 1;
    }
}
```

3.树的最小支配集(最少的点覆盖所有点)

 f[i][0]选i且i及i的子树都被覆盖了
 f[i][1]不选i且i被其儿子覆盖
 f[i][2]不选i且i被其父亲覆盖(儿子可选可不选)

```cpp
void dfs(int u, int fa){
    f[u][0] = 1; f[u][1] = f[u][2] = 0;
    bool ok = false;
    int tmp = inf32;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs(v, u);
        f[u][2] += min(f[v][1], f[v][0]);
        f[u][0] += min({f[v][0], f[v][1], f[v][2]});
        if (f[v][0] <= f[v][1]){
            ok = true;
            f[u][1] += f[v][0];
        }else{
            f[u][1] += f[v][1];
            tmp = min(tmp, f[v][0] - f[v][1]);
        }
    }
    if (!ok) f[u][1] += tmp;
}
```

4.树的最大独立集(选定的任意两点之间无边)

```cpp
function<void(int, int)> dfs = [&](int u, int fa)
{
    dp[u][1] = h[u];
    for (auto v : G[u])
    {
        if (v == fa)
            continue;
        dfs(v, u);
        dp[u][0] += max(dp[v][1], dp[v][0]);
        dp[u][1] += dp[v][0];
    }
};
```

5.树上背包(最多不超过 $m$ 条边)

```cpp
#include <bits/stdc++.h>
#include <cstring>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 1e2 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

vector<pair<int, int>> G[N];
int n, m, dp[N][N], sz[N], tmp[N];

void dfs(int u, int fa){
    sz[u] = 1;
    for (auto [v, w] : G[u]){
        if (v == fa) continue;
        dfs(v, u);
        memset(tmp, 0, sizeof tmp);
        for (int i = sz[u] - 1; i >= 0; --i){
            for (int j = sz[v] - 1; j >= 0; --j){
                int nxt = i + j + 1;
                if (nxt <= m){
                    tmp[nxt] = max(tmp[nxt], dp[u][i] + dp[v][j] + w);
                }
            }
        }
        sz[u] += sz[v];
        for (int i = 0; i <= sz[u] - 1; ++i) 
            dp[u][i] = max(dp[u][i], tmp[i]);
    }
}

void solve(){
    cin >> n >> m;
    for (int i = 1; i < n; ++i){
        int u, v, w;
        cin >> u >> v >> w;
        G[u].emplace_back(v, w);
        G[v].emplace_back(u, w);
    }
    dfs(1, 0);
    int ans = 0;
    for (int i = 1; i <= m; ++i){
        ans = max(ans, dp[1][i]);
    }
    cout << ans << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

6.2022CCPC-A(`树上背包`)
爱丽丝想在公园里找到她丢失的猫。

爱丽丝想在公园里找到她丢失的猫。

公园是一棵有根的树，由 $n$ 个顶点组成。顶点的编号从 $1$ 到 $n$ ，根顶点为 $1$ 。

爱丽丝现在位于顶点 $1$ 。她知道猫已经从顶点 $1$ 跑到了树的某片叶子上，而且没有顶点被访问超过一次。叶子是没有子顶点的顶点。

每个顶点上都有一个监视器。顶点 $i$ 上的监视器可以观察到猫是否访问了顶点 $i$ ，以及猫去了哪个顶点(如果顶点 $i$ 不是叶子)。爱丽丝需要花费 $a_i$ 秒来检查 $i_{th}$ 监视器的数据。

爱丽丝也可以自己搜索一些叶子。搜索 $i$ 个叶子需要 $t_i$ 秒。请注意， $i$ 是顶点的计数，而不是顶点的标签。

帮助爱丽丝确定要检查哪些监视器和搜索哪些树叶，以便唯一确定猫的位置，并尽可能减少所需的总时间。请注意，要检查的监视器和要搜索的树叶应该在一开始就决定好，之后不得更改。

找出最短的时间。

```cpp
#include <bits/stdc++.h>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e3 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

vector<int> G[N];
ll sz[N], a[N], t[N];
ll dp[N][N][2], g[N][N];
ll tmp1[N][2], tmp2[N];


void dfs(int u, int fa){
    if (G[u].size() == 1 && fa){
        sz[u] = 1;
        dp[u][0][0] = dp[u][1][1] = 0;
        dp[u][1][0] = a[u];
        return;
    }
    dp[u][0][0] = 0;
    g[u][0] = 0;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs(v, u);
        memset(tmp1, 0x3f, sizeof tmp1);
        memset(tmp2, 0x3f, sizeof tmp2);
        for (int i = sz[u]; i >= 0; --i){
            for (int j = sz[v]; j >= 0; --j){
                int nxt = i + j;
                tmp1[nxt][0] = min(tmp1[nxt][0], dp[u][i][0] + dp[v][j][0]);
                tmp1[nxt][1] = min({tmp1[nxt][1], dp[u][i][0] + dp[v][j][1], dp[u][i][1] + dp[v][j][0]});
                tmp2[nxt] = min(tmp2[nxt], g[u][i] + min(dp[v][j][0], dp[v][j][1]));
            }
        }
        sz[u] += sz[v];
        memcpy(dp[u], tmp1, sizeof tmp1);
        memcpy(g[u], tmp2, sizeof tmp2);
    }
    for (int i = 1; i <= sz[u]; ++i)
        dp[u][i][0] = min(dp[u][i][0], g[u][i] + a[u]);
}

void solve(){
    int n;
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    for (int i = 1; i <= n; ++i) cin >> t[i];
    for (int i = 1; i <= n; ++i) sz[i] = 0;
    for (int i = 1; i <= n; ++i) G[i].clear();
    for (int i = 1, u, v; i < n; ++i){
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    } 
    memset(dp, 0x3f, sizeof dp);
    memset(g, 0x3f, sizeof g);
    if (n == 1) {
        cout << 0 << endl;
        return;
    }
    ll ans = inf64;
    dfs(1, 0);
    for (int i = 1; i <= sz[1]; ++i){
        ans = min(ans, t[sz[1] - i] + min(dp[1][i][0], dp[1][i][1]));
    }
    cout << ans << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    cin >> t;
    while(t--) solve();
    return 0;
}
```

```cpp
#include <bits/stdc++.h>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e3 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

vector<int> G[N];
ll sz[N], a[N], t[N];
ll dp[N][N][2], g[N][N];

void dfs(int u, int fa){
    if (G[u].size() == 1 && fa){
        sz[u] = 1;
        dp[u][0][0] = dp[u][1][1] = 0;
        dp[u][1][0] = a[u];
        return;
    }
    dp[u][0][0] = 0;
    g[u][0] = 0;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs(v, u);
        for (int i = sz[u]; i >= 0; --i){
            for (int j = sz[v]; j >= 0; --j){
                int nxt = i + j;
                dp[u][nxt][0] = min(dp[u][nxt][0], dp[u][i][0] + dp[v][j][0]);
                dp[u][nxt][1] = min({dp[u][nxt][1], dp[u][i][0] + dp[v][j][1], dp[u][i][1] + dp[v][j][0]});
                g[u][nxt] = min(g[u][nxt], g[u][i] + min(dp[v][j][0], dp[v][j][1]));
            }
        }
        sz[u] += sz[v];
    }
    for (int i = 1; i <= sz[u]; ++i)
        dp[u][i][0] = min(dp[u][i][0], g[u][i] + a[u]);
}

void solve(){
    int n;
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    for (int i = 1; i <= n; ++i) cin >> t[i];
    for (int i = 1; i <= n; ++i) sz[i] = 0;
    for (int i = 1; i <= n; ++i) G[i].clear();
    for (int i = 1, u, v; i < n; ++i){
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    } 
    memset(dp, 0x3f, sizeof dp);
    memset(g, 0x3f, sizeof g);
    if (n == 1) {
        cout << 0 << endl;
        return;
    }
    ll ans = inf64;
    dfs(1, 0);
    for (int i = 1; i <= sz[1]; ++i){
        ans = min(ans, t[sz[1] - i] + min(dp[1][i][0], dp[1][i][1]));
    }
    cout << ans << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    cin >> t;
    while(t--) solve();
    return 0;
}
```

7.树联通点集(`换根`)

```cpp
#include <bits/stdc++.h>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 1e6 + 10;
const ll mod = 1e9 + 7;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

int n;
ll f[N], ans[N];
vector<int> G[N];

ll qmi(ll a, ll b = mod - 2){
    ll res = 1;
    while (b){
        if (b & 1) res = res * a % mod;
        a = a * a % mod;
        b >>= 1;
    }
    return res;
}

void dfs1(int u, int fa){
    f[u] = 1;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs1(v, u);
        f[u] = f[u] * (f[v] + 1) % mod;
    }
}

void dfs2(int u, int fa){
    if (!fa) ans[u] = f[u];
    else if ((f[u] + 1) % mod == 0){
        dfs1(u, u);
        ans[u] = f[u];
    }else ans[u] = (ans[fa] % mod * qmi(f[u] + 1) + 1) % mod * f[u] % mod;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs2(v, u);
    }
}

void solve(){
    cin >> n;
    for (int i = 1; i < n; ++i){
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    dfs1(1, 1);
    dfs2(1, 0);
    for (int i = 1; i <= n; ++i) cout << ans[i] << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

8.树划分联通块大小小于等于k(`树上背包`)

另一种背包写法

```cpp
#include <bits/stdc++.h>
#include <cstring>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e3 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

int n, k, sz[N];
vector<int> G[N];
ll dp[N][N], ans = 0; //dp[i][j] i 所在的联通块大小为 j

void dfs(int u, int fa){
    sz[u] = dp[u][1] = 1;
    for (auto v : G[u]){
        if (v == fa) continue;
        dfs(v, u);
        ll sum = 0;
        for (int i = 1; i <= sz[v] && i <= k; ++i) sum = (sum + dp[v][i]) % mod;
        for (int i = min(sz[u], k); i >= 1; --i){
            for (int j = min(sz[v], k); j >= 1; --j){
                int nxt = i + j;
                if (nxt <= k){ 
                    dp[u][nxt] = (dp[u][nxt] + dp[u][i] * dp[v][j]) % mod;
                }
            }
            dp[u][i] = (dp[u][i] * sum) % mod;
        }
        sz[u] += sz[v];
    }
}

void solve(){
    cin >> n >> k;
    for (int i = 1; i < n; ++i){
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    dfs(1, 0);
    for (int i = 1; i <= k && i <= sz[1]; ++i) ans = (ans + dp[1][i]) % mod;
    cout << ans << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

8.树上子链(`点权和最大`)

给定一棵树 T ，树 T 上每个点都有一个权值。
定义一颗树的子链的大小为：这个子链上所有结点的权值和 。
请在树 T 中找出一条最大的子链并输出。

```cpp
#include <bits/stdc++.h>
#define endl '\n'
#define pll pair<ll, ll>
#define tll tuple<ll, ll, ll>
#define ios ios::sync_with_stdio(false), cin.tie(0), cout.tie(0)
using namespace std;
typedef long long ll;
const ll maxn = 2e5 + 10;
const ll mod = 998244353;
vector<ll> G[maxn];
ll w[maxn], dis[maxn], ans = -1e18;

void solve(){
    int n;
    cin >> n;
    for (int i = 1; i <= n; ++i){
        cin >> w[i];
    }
    for (int i = 1; i <= n - 1; ++i){
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    function<void(int, int)> dfs = [&](int u, int fa){
        ll tmp = 0, mx1 = 0, mx2 = 0;
        for (auto v: G[u]){
            if (v == fa) continue;
            dfs(v, u);
            tmp = dis[v];
            if (tmp >= mx1){
                mx2 = mx1;
                mx1 = tmp;
            }else if (tmp >= mx2){
                mx2 = tmp;
            }
        }
        ans = max(ans, mx1 + mx2 + w[u]);
        dis[u] = mx1 + w[u];
    };
    dfs(1, 0);
    cout << ans << endl;
}

int main(){
    ios;
    int t = 1;
    //cin >> t;
    while(t--){
        solve();
    }
    return 0;
}
```

9.Nim Cheater(`树剖优化dp`)

```cpp
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 4e4 + 10;
constexpr int mod = 998244353;

using namespace std;

int dp[16390];
int f[32][16390];

void solve(){
    int n;
    cin >> n;
    vector<int> a(n + 1), b(n + 1), fa(n + 1), ask(n + 1), ans(n + 1);
    vector<vector<int>> G(n + 1);
    int lst = 0, rt = 0;
    for (int i = 1; i <= n; ++i) {
        string s;
        cin >> s;
        if (s[0] == 'A'){
            cin >> a[i] >> b[i];
            if (rt == 0) rt = i, lst = i;
            else{
                G[lst].push_back(i);
                fa[i] = lst;
                lst = i;
            }
            ask[i] = i;
        }else{
            lst = fa[lst];
            ask[i] = lst;
        }
    }
    vector<int> sz(n + 1), son(n + 1);

    function<void(int)> dfs1 = [&](int u) {
        sz[u] = 1;
        for (auto v : G[u]){
            dfs1(v);
            sz[u] += sz[v];
            if (sz[v] > sz[son[u]]) son[u] = v; 
        }
    };

    dfs1(rt);

    function<void(int, int, int)> dfs2 = [&](int u, int num, int sum){
        memcpy(dp, f[num], sizeof(dp));
        for (int i = 0; i < 16384; ++i){
            dp[i] = max(dp[i], f[num][i ^ a[u]] + b[u]);
        }
        ans[u] = sum - dp[0];
        memcpy(f[num], dp, sizeof(f[num]));
        for (auto v : G[u]){
            if (v == son[u]) continue;
            memcpy(f[num + 1], f[num], sizeof(f[num]));
            dfs2(v, num + 1, sum + b[v]); 
        }
        if (son[u])
            dfs2(son[u], num, sum + b[son[u]]);
    };

    memset(f, -0x3f, sizeof(f));
    f[0][0] = 0;
    dfs2(rt, 0, b[rt]);
    for (int i = 1; i <= n; ++i) cout << ans[ask[i]] << endl;
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

## dsu on tree ## 

1.点分树模版(`距离为k是否存在`)

```cpp
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 2e5 + 10;
constexpr int mod = 998244353;

using namespace std;

void solve() {
    int n, m;
    cin >> n >> m;
    vector<vector<pair<int, int>>> G(n + 1);
    for(int i = 1; i < n; i++) {
        ll u, v, w;
        cin >> u >> v >> w;
        G[u].emplace_back(v, w);
        G[v].emplace_back(u, w);
    }

    vector <int> q(m + 1), ans(m + 1);

    for(int i = 1; i <= m; i++) {
        cin >> q[i];
    }

    set <int> s;
    for(int i = 1; i <= m; i++) {
        s.insert(i);
    }

    vector <int> l(n + 1), r(n + 1), siz(n + 1), son(n + 1), id(n + 1);
    vector <ll> dis(n + 1);
    int idx = 0;

    auto dfs1 = [&](auto self, int now, int p) -> void {
        l[now] = ++idx, id[idx] = now;
        for(auto [u, w] : G[now]) {
            if(u == p) continue;
            dis[u] = dis[now] +  w;
            self(self, u, now);
            siz[now] += siz[u];
            if(siz[u] > siz[son[now]]) {
                son[now] = u;
            }
        }
        siz[now]++;
        r[now] = idx;
    };

    dfs1(dfs1, 1, 0);

    unordered_map<ll, int> cnt;

    auto upd = [&](int x, int k) {
        cnt[dis[x]] += k;
    };

    auto dfs2 = [&](auto self, int now, int p, int keep) -> void {
        for(auto [u, w] : G[now]) {
            if(u == p || u == son[now]) continue;
            self(self, u, now, 0);
        }

        if(son[now]) self(self, son[now], now, 1);

        vector <int> ww;
        for(auto i : s) {
            if(cnt[dis[now] + q[i]]) {
                ans[i] = 1;
                ww.push_back(i);
            }
        }

        while(!ww.empty()) {
            s.erase(ww.back());
            ww.pop_back();
        }

        upd(now, 1);

        for(auto [u, w] : G[now]) {
            if(u == p || u == son[now]) continue;
            for(int i = l[u]; i <= r[u]; i++) {
                for(auto j : s) {
                    ll res = dis[now] + dis[now] + q[j] - dis[id[i]];
                    if(cnt[res]) {
                        ans[j] = 1;
                        ww.push_back(j);
                    }
                }
            }

            while(!ww.empty()) {
                s.erase(ww.back());
                ww.pop_back();
            }

            for(int i = l[u]; i <= r[u]; i++) {
                upd(id[i], 1);
            }
        }

        if(!keep) {
            for(int i = l[now]; i <= r[now]; i++) {
                upd(id[i], -1);
            }
        }
    };

    dfs2(dfs2, 1, 0, 1);

    for(int i = 1; i <= m; i++) {
        cout << (ans[i] ? "AYE" : "NAY") << endl;
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

2.CF600E(`纯板子`)

```cpp
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 2e5 + 10;
constexpr int mod = 998244353;

using namespace std;

int cnt[N];

void solve(){
    int n;
    cin >> n;
    vector<int> col(n + 1);
    for (int i = 1; i <= n; ++i){
        cin >> col[i];
    }
    vector<vector<int>> G(n + 1);
    for (int i = 1; i < n; ++i){
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }

    vector<int> l(n + 1), r(n + 1), rnk(n + 1), sz(n + 1), son(n + 1);
    int tot = 0;

    function<void(int, int)> dfs1 = [&](int u, int fa){
        sz[u] = 1;
        l[u] = ++tot;
        rnk[tot] = u;
        for (auto v : G[u]){
            if (v == fa) continue;
            dfs1(v, u);
            sz[u] += sz[v];
            if (sz[v] > sz[son[u]]) son[u] = v;
        }
        r[u] = tot;
    };

    dfs1(1, 0);

    vector<ll> ans(n + 1);
    ll res = 0, mx = -1;

    auto modify = [&] (int x){
        cnt[col[x]]++;
        if (cnt[col[x]] == mx) res += col[x];
        else if (cnt[col[x]] > mx) {
            mx = cnt[col[x]];
            res = col[x];
        }
    };

    function<void(int, int, int)> dfs2 = [&](int u, int fa, int keep){
        for (auto v : G[u]){
            if (v == fa || v == son[u]) continue;
            dfs2(v, u, 0);
        }
        if (son[u]) dfs2(son[u], u, 1);

        modify(u);
        
        for (auto v : G[u]){
            if (v == fa || v == son[u]) continue;
            for (int i = l[v]; i <= r[v]; ++i){
                modify(rnk[i]);
            }
        }

        ans[u] = res;
        
        if (!keep){
            for (int i = l[u]; i <= r[u]; ++i){ 
                cnt[col[rnk[i]]]--;
            }
            mx = -1;
            res = 0;
        }
    };

    dfs2(1, 0, 1);

    for (int i = 1; i <= n; ++i) cout << ans[i] << " \n"[i == n];
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


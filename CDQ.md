## CDQ 解决高维偏序 ##

1.HDU2024暑期多校1007 \
$i < j$ \
$f[i] < f[j]$ \
$g[i] < g[j]$ \
满足上面三个条件则 $i$ 向 $j$ 建边，计算每个点的出度

```cpp
#include <algorithm>
#include <bits/stdc++.h>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e5 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

int tr[N];
int lowbit(int x) {return x & -x;}
void add(int p, int val){
    for (; p < N; p += lowbit(p)){
        tr[p] += val;
    }
}
int query(int p){
    int res = 0;
    for (; p; p -= lowbit(p)){
        res += tr[p];
    }
    return res;
}

int query(int p1, int p2){
    return query(p2) - query(p1 - 1);
}

struct Query{
    int f, g, idx;
}node[N];

ll f_[N], g_[N];
ll n;
ll sum[N];

Query tmp[N];

void cdq1(int l, int r){
    int mid = (l + r) >> 1;
    if (l == r) {
        node[l].f = n - f_[l] + 1;
        node[l].g = n - g_[l] + 1;
        node[l].idx = l; 
        return;
    }
    cdq1(l, mid);
    cdq1(mid + 1, r);
    int cnt = l, f1 = l, f2 = mid + 1;
    while (f1 <= mid && f2 <= r){
        if (node[f1].f <= node[f2].f) {
            sum[node[f1].idx] += query(node[f1].g - 1);
            tmp[cnt++] = node[f1++];
        }else{
            add(node[f2].g, 1);
            tmp[cnt++] = node[f2++];
        }
    }
    while (f1 <= mid){
        sum[node[f1].idx] += query(node[f1].g - 1);
        tmp[cnt++] = node[f1++];
    }
    for (int i = mid + 1; i < f2; ++i) add(node[i].g, -1);
    while (f2 <= r){
        tmp[cnt++] = node[f2++];
    }
    for (int i = l; i <= r; ++i){
        node[i] = tmp[i];
    }
}

void cdq2(int l, int r){
    int mid = (l + r) >> 1;
    if (l == r) {
        node[l].f = f_[l];
        node[l].g = g_[l];
        node[l].idx = l; 
        return;
    }
    cdq2(l, mid);
    cdq2(mid + 1, r);
    int cnt = l, f1 = l, f2 = mid + 1;
    while (f1 <= mid && f2 <= r){
        if (node[f1].f < node[f2].f) {
            add(node[f1].g, 1);
            tmp[cnt++] = node[f1++];
        }else{
            sum[node[f2].idx] -= query(node[f2].g - 1);
            tmp[cnt++] = node[f2++];
        }
    }
    while(f2 <= r) {
        sum[node[f2].idx] -= query(node[f2].g - 1);
        tmp[cnt++] = node[f2++];
    }
	for(int i = l; i < f1; i++) add(node[i].g, -1);
	while(f1 <= mid){
        tmp[cnt++] = node[f1++];
    }
    for (int i = l; i <= r; ++i){
        node[i] = tmp[i];
    }
}

void solve(){
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> f_[i];
    for (int i = 1; i <= n; ++i) cin >> g_[i];
    ll ans = 1ll * n * (n - 1) * (n - 2) / 6;
    for (int i = 1; i <= n; ++i) sum[i] = i - 1;
    cdq1(1, n);
    cdq2(1, n);
    // for (int i = 1; i <= n; ++i){
    //     cout << sum[i] << " \n"[i == n];
    // }
    for (int i = 1; i <= n; ++i) ans = ans - sum[i] * (sum[i] - 1) / 2;
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

2.CF1045G

火星上有$N$个机器人排成一行，第$i$个机器人的位置为$x_{i}$，视野为$r_{i}$，智商为$q_{i}$。我们认为第$i$个机器人可以看到的位置是$[x_{i}-r_{i},x_{i}+r_{i}]$。	
如果一对机器人相互可以看到，且它们的智商$q_{i}$的差距**不大于**$K$，那么它们会开始聊天。	
为了防止它们吵起来，请计算有多少对机器人可能会聊天。

```cpp
#include <algorithm>
#include <bits/stdc++.h>
#define endl '\n'
#define int ll
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 1e5 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

struct Query{
    int pos, R, l, r;
    int q;
}qry[N];

int tr[N << 2], mx, n, k, ans = 0;

int lowbit(int x) {return x & -x;}

void add(int p, int val){
    for (int i = p; i <= mx; i += lowbit(i)) tr[i] += val;
}

int query(int p){
    int res = 0;
    for (int i = p; i; i -= lowbit(i)) res += tr[i];
    return res;
}

int query(int p1, int p2){
    return query(p2) - query(p1 - 1);
}

bool cmplen(const Query& x, const Query& y){
    return x.R > y.R;
}

unordered_map<int, int> mp;
set<int> t;

Query tmp[N];

void merge(int l, int mid, int r){
    int cnt = l;
    int i = l, j = mid + 1;
    while (i <= mid && j <= r) tmp[cnt++] = qry[i].q <= qry[j].q ? qry[i++] : qry[j++];
    while (i <= mid) tmp[cnt++] = qry[i++];
    while (j <= r) tmp[cnt++] = qry[j++];
    for (int i = l; i <= r; ++i) qry[i] = tmp[i];
}

void cdq(int l, int r){
    int mid = (l + r) >> 1;
    if (l == r) return;
    cdq(l, mid);
    cdq(mid + 1 ,r);
    int cl = l, cr = l - 1;
    for (int i = mid + 1; i <= r; ++i){
        auto [pos, R, ql, qr, q] = qry[i];
        while (cl <= mid && q - k > qry[cl].q) add(qry[cl++].pos, -1);
        while (cr < mid && qry[cr + 1].q <= q + k) add(qry[++cr].pos, 1);
        ans += query(ql, qr);
    }
    for (int i = cl; i <= cr; ++i) add(qry[i].pos, -1);
    merge(l, mid, r);// 对 q 排序满足第二个偏序条件
}

void solve(){   
    cin >> n >> k;
    for (int i = 1; i <= n; ++i){
        auto &[pos, R, _1, _2, q] = qry[i];
        cin >> pos >> R >> q;
        t.insert(pos);
        t.insert(pos - R);
        t.insert(pos + R);
    }
    for (auto v : t) mp[v] = ++mx;
    for (int i = 1; i <= n; ++i){
        qry[i].l = mp[qry[i].pos - qry[i].R];
        qry[i].r = mp[qry[i].pos + qry[i].R];
        qry[i].pos = mp[qry[i].pos];
    }
    // 对 R 排序满足第一个偏序条件
    sort(qry + 1, qry + n + 1, cmplen);
    cdq(1, n);
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

3.陌上开花('纯板子')

有 $ n $ 个元素，第 $ i $ 个元素有 $ a_i,b_i,c_i $ 三个属性，设 $ f(i) $ 表示满足 $ a_j \leq a_i $ 且 $ b_j \leq b_i $ 且 $ c_j \leq c_i $ 且 $ j \ne i $ 的 $j$ 的数量。

```cpp
#include <bits/stdc++.h>
#include <vector>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 2e5 + 10;
const ll mod = 998244353;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;

struct Query{
    int a, b, c;
    int w, ans;
}node[N], qry[N];

struct BIT{
    vector<int> tr;
    void init(int n){
        vector<int>(n + 1).swap(tr);
    }
    int lowbit(int x) {return x & -x;}
    void add(int x, int val){
        for (; x < tr.size(); x += lowbit(x))
            tr[x] += val;
    }
    int query(int x){
        int res = 0;
        for (; x; x -= lowbit(x)){
            res += tr[x];
        }
        return res;
    }
}T;

bool cmpa(const Query& u, const Query& v){
    if (u.a == v.a){
        if (u.b == v.b)
            return u.c < v.c;
        return u.b < v.b;
    }
    return u.a < v.a;
}

bool cmpb(const Query& u, const Query& v){
    if (u.b == v.b){
        return u.c < v.c;
    }
    return u.b < v.b;
}

Query tmp[N];
ll cnt[N];

void cdq(int l, int r){
    if (l == r) return;
    int mid = (l + r) >> 1;
    cdq(l, mid);
    cdq(mid + 1, r);
    sort(qry + l, qry + mid + 1, cmpb);
    sort(qry + mid + 1, qry + r + 1, cmpb);
    int f2 = mid + 1, f1 = l;
    for (; f2 <= r; ++f2){
        while (qry[f1].b <= qry[f2].b && f1 <= mid){
            T.add(qry[f1].c, qry[f1].w);
            f1++;
        }
        qry[f2].ans += T.query(qry[f2].c);
    }
    for (int i = l; i < f1; ++i) T.add(qry[i].c, -qry[i].w);
}

void solve(){
    int n_, k, n = 0;
    cin >> n_ >> k;
    T.init(k);
    for (int i = 1; i <= n_; ++i){
        auto &[a, b, c, w, ans] = node[i];
        cin >> a >> b >> c;
    }
    sort(node + 1, node + n_ + 1, cmpa);
    int c = 0;
    for (int i = 1; i <= n_; ++i){
        c++;
        if (node[i].a != node[i + 1].a || node[i].b != node[i + 1].b || node[i].c != node[i + 1].c){
            qry[++n] = node[i];
            qry[n].w = c;
            c = 0;
        }
    }
    cdq(1, n);
    for (int i = 1; i <= n; ++i){
        cnt[qry[i].ans + qry[i].w - 1] += qry[i].w;
    }
    for (int i = 0; i < n_; ++i) cout << cnt[i] << endl;
}

signed main(){
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

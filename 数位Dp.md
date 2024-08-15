## 数位DP ##

1.XOR SUM

路易斯喜欢整数。他将非负整数序列 $A = [a_1, a_2 ,\ldots, a_k]$ 的值定义为以下公式（ $\oplus$ 表示位排他性-或）：

$$
\sum \limits_{i = 1}^k \sum \limits_{j = 1}^{i - 1} a_i \oplus a_j
$$

他想知道有多少个不同的序列 $A$ 满足以下条件：

- $A$ 的长度是 $k$ 。
- $A$ 的值是 $n$ 。
- $0\le a_i\le m$ ( $1\le i\le k$ ).

当且仅当存在一个索引 $i$ 时， $[a_1, \dots, a_k]$ 、 $[b_1, \dots, b_k]$ 这两个序列被认为是不同的，即 $a_i \neq b_i$ 。告诉路易斯答案模块 $10^9 + 7$ 。


输入一行包含三个整数 $n$ , $m$ , $k$ , ( $0 \leq n \leq 10^{15}$ , $0 \leq m \leq 10^{12}$ , $1 \leq k \leq 18$ )。

```cpp
#include <bits/stdc++.h>
#define endl '\n'
using namespace std;
typedef long double db;
typedef long long ll;

const ll N = 21;
const ll mod = 1e9 + 7;
const ll inf32 = 0x3f3f3f3f;
const ll inf64 = 5e18;
int C[N][N];

void init(int n){
    int i, j;
    for (C[0][0] = i = 1; i <= n; ++i)
        for (C[i][0] = j = 1; j <= i; ++j)
            C[i][j] = (C[i - 1][j] + C[i - 1][j - 1]) % mod;
}

ll n, m, k;
unordered_map<ll, ll> f[N << 1][N];
ll dp(int now, int lim, ll sum){
    if (sum > n) return 0;
    if (sum + (k >> 1ll) * (k + 1 >> 1ll) * ((1ll << now + 1) - 1) < n) return 0ll;
    if (now < 0) return sum == n;
    if (f[now][lim].count(sum)) return f[now][lim][sum];
    ll res = 0;
    if (m >> now & 1){
        for (int i = 0; i <= lim; ++i) //之前卡上界的个数
            for (int j = 0; j <= k - lim; ++j)
                (res += C[lim][i] * C[k - lim][j] % mod * dp(now - 1, i ,sum + (i + j) * (k - i - j) * (1ll << now)) % mod) %= mod;
    }else{
        for (int j = 0; j <= k - lim; ++j){
            (res += C[lim][0] * C[k - lim][j] % mod * dp(now - 1, lim, sum + j * (k - j) * (1ll << now)) % mod) %= mod;
        }
    }
    return f[now][lim][sum] = res;
}

void solve(){
    cin >> n >> m >> k;
    init(k);    
    ll ans = dp(39, k, 0);
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
## 线性dp ##

#### 最长单峰子序列 ####

```cpp
#include <bits/stdc++.h>
#define endl '\n'

using ll = long long;

constexpr int N = 2e5 + 10;
constexpr int mod = 998244353;

using namespace std;

struct SegmentTree
{
	ll tr[N << 2];

	void push_up(int u) {
		tr[u] = max(tr[u << 1], tr[u << 1 | 1]);
	}

	void modify(int u, int l, int r, int pos, ll val) {
		if (l == r) {
			tr[u] = val;
			return;
		}
		int mid = (l + r) >> 1;
		if (pos <= mid) modify(u << 1, l, mid, pos, val);
		else modify(u << 1 | 1, mid + 1, r, pos, val);
		push_up(u);
	}

	ll query_max(int u, int l, int r, int ql, int qr) {
		if (ql <= l && r <= qr) return tr[u];
		int mid = (l + r) >> 1;
		ll ans = 0;
		if (ql <= mid) {
			ll tmp = query_max(u << 1, l, mid, ql, qr);
			ans = max(ans, tmp);
		}
		if (qr > mid) {
			ll tmp = query_max(u << 1 | 1, mid + 1, r, ql, qr);
			ans = max(ans, tmp);
		}
		return ans;
	}
}T0, T1;

ll a[N];

void solve(){
	int n, m;
	cin >> n >> m;
	for (int i = 1; i <= n; ++i) {
		int l, r;
		cin >> l >> r;
		a[l]++;
		a[r + 1]--;
	}
	for (int i = 1; i <= m; ++i) a[i] += a[i - 1];
	for (int i = 1; i <= m; ++i) {
		ll v = T0.query_max(1, 0, n, 0, a[i]);
		T0.modify(1, 0, n, a[i], v + 1);
		T1.modify(1, 0, n, a[i], max(T1.query_max(1, 0, n, a[i], n), v) + 1);
	}
	cout << (T0.query_max(1, 0, n, 0, n), T1.query_max(1, 0, n, 0, n)) << endl;
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
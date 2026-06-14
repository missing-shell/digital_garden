### 未优化
```c++
#include<iostream>
#include<algorithm>
#include<string>

using namespace std;

const int N = 1e5 + 10;
int n, m;
int p[N], cnt[N];

int find(int x)
{
	if (p[x] != x) {
		p[x] = find(p[x]);
	}
	return p[x];
}

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++)
	{
		p[i] = i;
		cnt[i] = 1;
	}
	while (m--) {
		string s;
		int a, b;
		cin >> s;
		if (s == "C") {//此处可能出现严重bug!!!!!!!!!!
			cin >> a >> b;
			
			if (find(a) != find(b)) {  
				cnt[find(b)] += cnt[find(a)];
 				p[find(a)] = find(b);//此处必须先计算再合并，cnt会重新调用新的find,此时已合并为同一集合

			}
		}
		else if (s == "Q1") {
			cin >> a >> b;
			if (find(a) == find(b)) {
				cout << "Yes" << endl;
			}
			else {
				cout << "No" << endl;
			}
		}
		else {

			cin >> a;
			cout << cnt[find(a)] << endl;
		}

	}
	return 0;
}
```

### 对 if(s == “C")优化

^872140

```c++
if (s == "C") {
			cin >> a >> b;
		    int x = find(a), y = find(b);//提前记录两集合根节点，避免p[x]=y对下一步操作造成影响
			if (x != y) {
			    p[x]=y;
				cnt[y] += cnt[x];
			}
		}
```
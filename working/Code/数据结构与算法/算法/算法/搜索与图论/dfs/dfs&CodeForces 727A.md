 ```c++
 #include<iostream>
#include<algorithm>
using namespace std;
const int N = 5000000;
int ans[N];
int n, m;
int flag=0;//记录状态，yes  or  no 
int res=0;//记录次数
 
void dfs(int x,int cnt) {
	if (x == m) {
		flag = 1;
		res=cnt;
		return;
	}
	if (x > m) {
		return;
	}
	ans[cnt] = x * 10 + 1;
	if (x <= 1e8) {
		dfs(x * 10 + 1, cnt + 1);
	}
	if (flag) {  //任然有疑问？
		return;
	}
	ans[cnt] = x * 2;
	dfs(x * 2, cnt + 1);
	 

}
int main()
{
	cin >> n >> m;
	ans[0] = n;
	dfs(n,1);
	if (flag) {
		cout << "YES" << endl << res << endl;
		for (int i = 0; i < res; i++) {
			printf("%d%c", ans[i], i == res - 1 ? '\n' : ' ');
		}
	}
	else {
		cout << "NO" << endl;
	}
	return 0;
}
```
 ```c++
 /*同全排列方式递归，后回溯*/
#include<iostream>
using namespace std;
const int N = 10;
char g[N][N];
bool  col[N],dg[2*N],udg[2*N];//状态数组
int n;
void dfs(int cnt) {
	if (cnt == n) {
		for (int i = 0; i < n; i++)puts(g[i]);
		puts("");
		return;
	}
	for (int i = 0; i <  n; i++) {
		if (!col[i] && !dg[cnt + i] && !udg[cnt - i + n]) {
			g[cnt][i] = 'Q';
			col[i] = dg[cnt + i] = udg[cnt - i + n] = true;
			dfs(cnt + 1);
			col[i] = dg[cnt + i] = udg[cnt - i + n] = false;
			g[cnt][i] = '.';
		}
	}

}
int main()
{
	scanf("%d", &n);
	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++) {
			g[i][j] = '.';
		}
	dfs(0);
	return 0;
}
```
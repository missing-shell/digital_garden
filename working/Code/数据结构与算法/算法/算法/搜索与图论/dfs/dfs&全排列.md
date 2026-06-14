```c++
#include<iostream>
using namespace std;
const int N = 10;
int path[N];//每一次只保存每一个分支
bool st[N];//状态数组
int n;
void dfs(int u) {//第几个数字，一共几个数字
	if (u == n) { //递归结束
		for (int i = 0; i < n; i++) {
			printf("%d%c", path[i], i == n - 1 ? '\n' : ' ');
		}
	}
	for (int i = 1; i <= n; i++) {
		if (!st[i]) {
			path[u] = i;
			st[i] = true;//i被用过
			dfs(u + 1);//下一层
			st[i] = false;//回复现场，以便回溯
		}
	}
}
int main()
{
	scanf("%d", &n);
	dfs(0);
	return 0;
}
```
 ```c++
 #include<iostream>
#include<algorithm>
#include<cstring>
#include<cstdio>

using namespace std;

const int N = 1e5 + 10,M=N*2;//无向图，两链表
int n;
int h[N], e[M], ne[M], idx;
int ans = N;//全局答案
bool st[N];//每个点都只搜一次

void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int dfs(int cnt)//返回以cnt为根节点的子树中结点数量
{
	st[cnt] = true;
	int size = 0, sum = 0;
	for (int i = h[cnt]; i ！= -1; i = ne[i]) {
		int j = e[i];
		if (st[j]) {
			continue;
		}
		int s = dfs(j);
		size = max(size, s);//找到以j为根节点的子树的结点数量
		sum += s;//表示以cnt为根节点的子树的结点的数量

	}
	size = max(size, n - sum - 1);
	ans = min(ans, size);
	return sum + 1;
}
int main()
{
	scanf_s("%d", &n);
	memset(h, -1, sizeof h);

	for (int i = 0; i < n - 1; i++) {
		int a, b;
		scanf_s("%d%d", &a, &b);
		add(a, b);//无向图，建立两个链表
		add(b, a);
	}
	dfs(1);//从任意一点开始搜
	printf("%d\n", ans);
	return 0;
}
```
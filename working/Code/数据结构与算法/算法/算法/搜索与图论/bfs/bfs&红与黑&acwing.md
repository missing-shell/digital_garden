**同[[bfs&走迷宫&acwing]],不过需要特别注意犯的错误**!!

此题也可使用dfs

## bfs
```c++
#include<iostream>
#include<cstring>
#include<algorithm>
#include<queue>
using namespace std;

typedef pair<int, int>PII;
const int N = 30;
int  m, n;
char arr[N][N];
bool st[N][N];
//int sum=1;
int bfs(int a,int b)
{
	int sum = 1;//只是累加记录返回值，没有必要定义成全局变量，就不要定义
	queue<PII>ans;
	memset(st,0, sizeof st);
	st[a][b] = true;
	ans.push({ a,b });
	int dx[4] = { 0,1,0,-1 }, dy[4] = { 1,0,-1,0 };
	while (ans.size()) {
		PII t = ans.front();
		ans.pop();
		for (int i = 0; i < 4; i++) 
		{
			int x = t.first + dx[i], y = t.second + dy[i];
			if (x < 0 || x >= n || y < 0 || y >= m)continue;
			if (st[x][y])continue;
			if (arr[x][y] != '.')continue;
			st[x][y] = true;
			ans.push({ x,y });
			sum++;
		}
	}
	return sum;
}

int main()
{
	while (~scanf_s("%d%d", &m, &n)&&n|m) {
		int x1=0, y1=0;
		for(int i=0;i<n;i++)
			for (int j = 0; j < m; j++) {
				cin >> arr[i][j];
				if (arr[i][j] == '@') {
					 x1 = i, y1 = j;
				}
			}
		//int t = bfs(x1, y1); 调试代码时此处增加了一处错误，访问了两次bfs，又因sum为全局变量，导致结果错误
		cout << bfs(x1,y1) << endl;
	}
	return 0;
}

```

## dfs

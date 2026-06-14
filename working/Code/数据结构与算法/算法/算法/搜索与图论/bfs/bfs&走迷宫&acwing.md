```c++
#include<iostream>
#include<cstring>
#include<algorithm>
using namespace std;
const int N = 1010;
typedef pair<int, int >PII; //模拟队列
int g[N][N]; //存储地图上的值
int d[N][N];//记录是否走过该点，以及走到该点所需步数
int n, m;
PII  q[N * N];//定义队列，
int bfs()
{
	q[0] = { 0,0 };//起始点
	memset(d, -1, sizeof d);
	d[0][0] = 0;//走过第一个点
	int hh = 0, tt = 0;
	int dx[4] = { -1,0,1,0 }, dy[4] = { 0,1,0,-1 };//上下左右四个方向
	while (hh <= tt) {//队列非空
		auto t = q[hh++];
		for (int i = 0; i < 4; i++) {//对每个点依次遍历四个方向的情况
			int x = t.first + dx[i], y = t.second + dy[i];
			if (x >= 0 && x < n && y >= 0 && y < m && g[x][y] == 0 && d[x][y] == -1) {//易错：y<n
				d[x][y] = d[t.first][t.second] + 1;
				q[++tt] = { x,y };
			}
		}
		 
	}
	return d[n - 1][m - 1];
}
int main()
{
	scanf_s ("%d%d", &n, &m);
	for(int i=0;i<n;i++)
		for (int j = 0; j < m; j++) {
			scanf_s ("%d", &g[i][j]);
		}
	 
	cout <<bfs() << endl;//一定不要忘记bfs的括号[[注意事项#^8c3c8f]]
	return 0;}```

  
```c++
int bfs()
{
	queue<PII>ans;
	memset(d, -1, sizeof d);
	d[0][0] = 0;
	ans.push({ 0,0 });
	int dx[4] = { -1,0,1,0 }, dy[4] = { 0,1,0,-1 };
	while (ans.size()) {
		auto t = ans.front();
		ans.pop();
		for (int i = 0; i < 4; i++) {
			int x = t.first + dx[i], y = t.second + dy[i];
			if (x>=0 && x < n && y >= 0 && y < m && g[x][y] == 0 && d[x][y] == -1) {
				d[x][y] = d[t.first][t.second] + 1;
				ans.push({ x,y });
			}
		}

	}
	return d[n - 1][m - 1];
}
```
 
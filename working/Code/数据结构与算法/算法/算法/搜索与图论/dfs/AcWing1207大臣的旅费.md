# DFS BFS 树型DP
题目内容：
	城市编号：`1~n`  
	公路`n-1`条
	双向公路：无向图
	最大连通子图
	求两点间最长距离
	花费可通过递归或循环求得  `d+=(d+1)`
	等差数列，首项为11，公差为1，`s(n)=a*n+(n*(n-1)*d)/2`
	样例数据为`4-2-5`,总长为9，结果为135

- 参考及拓展：[秦淮岸灯火阑珊](https://www.acwing.com/blog/content/319/)
### 树的直径 
在一棵树中，每一条边都有**权值**，树中的**两个点之间的距离**，定义为连接两点的路径上**边权之和**，那么树上最远的两个点，他们之间的距离，就被称之为，**树的直径**。

树的直径的别称，**树的最长链。**

请注意：树的直径，还可以认为是一条路径，不一定是只是一个数值。
## `dfs`
- 通过深度优先遍历找到与x的最远距离的点y
- 再通过深度优先遍历找到与y的最远距离

- `注意：`递归函数需要记录上一结点`father`
- 时间复杂度  $O(n)$

```c++
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 1e5 + 10,M=N*2;

int h[N], e[M], ne[M], w[M], idx;

int dist[N];
int n ;
void add(int a, int b, int c)
{
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dfs(int u,int father,int distance)
{
    dist[u] = distance;
    for (int i = h[u]; i != -1;i=ne[i])
    {
        int j = e[i];
        if(j!=father)
        {
            dfs(j, u, distance + w[i]);
        }

    }
}

int main()
{
    int n;
    cin >> n;
    memset(h, -1, sizeof(h));

    for (int i = 0; i < n; i++)
    {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
        add(b, a, c);
    }
    dfs(1, -1, 0);
    int u = 1;
    for (int i = 2; i <= n;i++)
    {
        if(dist[u]<dist[i])
        {
            u = i;
        }
    }
    dfs(u, -1, 0);
    for (int i = 1; i <= n;i++)
    {
        if(dist[u]<dist[i])
        {
            u = i;
        }
    }
    long long int res = dist[u];

    res = res * 11 + res * (res - 1) / 2;

    cout << res << endl;
    return 0;
}
```
- 重点
	如何判断已经形成环--如果能围成环，那么最后画的线的两个端点**必然已经在一个连通块内**

	判断是否在一个连通块内--并查集

- 地图是二维的，如何存点
	`ans[i][j]`对应下标`n*i+j`

	因为点的坐标是从`(1,1)`开始的，所以需要将`p[N]`的空间多开`2n`个，或者在输入坐标后`x--,y--`,将其映射到正常坐标系，此时`p[N]`初始化是从`0`开始

```c++
#include <iostream>
#include <algorithm>
#include <string>

using namespace std;

const int N = 210;

int p[N * N];

int find(int x)
{
    if (p[x] != x)
    {
        p[x] = find(p[x]);
    }
    return p[x];
}

int main()
{
    int n, m;
    cin >> n >> m;
    // 如果不将输入减一（a--,b--),则需确保p[N]的空间要多开
    for (int i = 1; i <= (n + 1) * (n + 1); i++)
    {
        p[i] = i;
    }

    for (int i = 1; i <= m; i++)
    {
        int a, b;
        string str;
        cin >> a >> b >> str;
        int x, y;
        if ("D" == str)
        {
            x = a + 1, y = b;
        }
        else
        {
            x = a, y = b + 1;
        }
        int oldd = find(a * n + b);
        int neww = find(x * n + y);

        if (oldd == neww)
        {
            cout << i << endl;
            return 0;
        }
        else
        {
            p[oldd] = neww;
        }
    }
    cout << "draw" << endl;
    return 0;
}
```
# 并查集  BFS

## 并查集
### 错误版本
- 尝试遍历`z[i]`,验证是否在`0~h`间连通。思路没有错误，但是因为`h`和`z[i]`的取值范围为`0~1e9`，所以无法实现
- 应该使用结构体，并遍历`0~n`
```c++
#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

typedef long long int LL;

const int N = 1e3 + 10;

int p[N];

LL dist(int a, int b, int c, int x, int y, int z)
{
    LL ret = (LL)(sqrt(pow(a - x, 2) + pow(b - y, 2) + pow(c - z, 2)));
    return ret;
}

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
    int t;
    cin >> t;

    while (t--)
    {
        int n, h, r;
        cin >> n >> h >> r;

        for (int i = 0; i <= h; i++)
        {
            p[i] = i;
        }

        int x[N], y[N], z[N];
        int zmin = 1e9, zmax = 0;

        for (int i = 0; i < n; i++)
        {
            cin >> x[i] >> y[i] >> z[i];
            zmin = min(zmin, z[i]);
            zmax = max(zmax, z[i]);
        }
        if (zmin <= r)
        {
            p[find(0)] = find(zmin);
        }
        if (zmax + r >= h)
        {
            p[find(zmax)] = find(h);
        }

        for (int i = 0; i < n; i++)
        {
            for (int j = i + 1; j < n; j++)
            {
                if (dist(x[i], y[i], z[i], x[j], y[j], z[j]) <= r * 2)
                {
                    p[find(z[i])] = find(z[j]);
                }
            }
        }
        if (find(0) == find(h))
        {
            puts("Yes");
        }
        else
        {
            puts("No");
        }
    }
    return 0;
}
```
### 改正

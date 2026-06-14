# 单调栈
前置题 131. 直方图中最大的矩形，相同题 152. 城市游戏。

## 思路
用`h[i][j]`表示第`j`列从第`i`行向上最多有多少块连续的未被破坏的土地，可从上到下递推得到。枚举矩形下边界，对于当前边界，每一列向上连续未被破坏的土地构成一个直方图，问题等价于在直方图中求出最大矩形。枚举直方图的每一列作为高，矩形左右边界最多延申到最近的比它低的一列，可通过单调栈求出。
![[AcWing1413矩形牛棚.png]]

- 枚举下边界，用两个单调栈分别取求左右两个第一个长度低于这一列的下标，然后`(r-l-1) * 高度` 便得到一个矩形面积
- 时间复杂度为o($n^2$)
### 关键点--抽象出高度
```c++
for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (!ans[i][j])
            {
                h[i][j] = h[i - 1][j] + 1;//计算高度
            }
        }
    }
```


## code
```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 3e3 + 10;

bool ans[N][N];
int h[N][N];
int q[N], tt = 0;
int l[N], r[N];

int n, m;

int work(int x)
{
    h[x][0] = h[x][m + 1] = -1;
    // 左边界
    tt = 0;
    q[++tt] = 0;

    for (int i = 1; i <= m; i++)
    {
        while ( tt && h[x][q[tt]] >= h[x][i])
        {
            tt--;
        }
        l[i] = q[tt];//左边最近最大的下标
        q[++tt] = i;
    }

    // 右边界
    tt = 0;
    q[++tt] = m + 1;//初始值，从右向左遍历
    for (int j = m; j >= 1; j--)
    {
        while ( tt && h[x][q[tt]] >= h[x][j])
        {
            tt--;
        }
        r[j] = q[tt];//右边最近最大的下标
        q[++tt] = j;
    }
    int ret = 0;
    for (int i = 1; i <= m; i++)
    {
        ret = max(ret, (r[i] - l[i] - 1) * h[x][i]);
    }
    return ret;
}
int main()
{
    int p;
    cin >> n >> m >> p;

    while (p--)
    {
        int x, y;
        cin >> x >> y;
        ans[x][y] = true;
    }
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (!ans[i][j])
            {
                h[i][j] = h[i - 1][j] + 1;//计算高度
            }
        }
    }

    int res = 0;
    for (int i = 1; i <= n; i++)//枚举下边界
    {
        res = max(res, work(i));
    }

    cout << res << endl;

    return 0;
}
```
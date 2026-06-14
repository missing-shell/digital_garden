#单调队列  #滑动窗口

## 滑动窗口

### code

- 本题等价于二维滑动窗口，分别对每一行和每一列进行滑动窗口
- 需要一个`s[N][N]`作为临时数组，存储每一行或者每一列的滑动窗口结果

```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

typedef long long int LL;

const int N = 1e3 + 10, MOD = 998244353;

int ans[N][N],s[N][N], s_min[N][N], s_max[N][N];

int q[N], hh = 0, tt = -1;

int main()
{
    ios::sync_with_stdio(0),cin.tie(0), cout.tie(0);

    int n, m, a, b;
    cin >> n >> m >> a >> b;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            cin >> ans[i][j];
        }
    }
    // 最小值
    for (int i = 0; i < n; i++)
    {
        hh = 0, tt = -1;
        for (int j = 0; j < m; j++)
        {
            while (hh <= tt && j - b + 1 > q[hh])
            {
                hh++;
            }
            while (hh <= tt && ans[i][q[tt]] >= ans[i][j])
            {
                tt--;
            }
            q[++tt] = j;
            s[i][j] = ans[i][q[hh]];
        }
    }
    for (int j = 0; j < m;j++)
    {
        hh = 0, tt = -1;
        for (int i = 0; i < n;i++)
        {
            while(hh<=tt&&i-a+1>q[hh])
            {
                hh++;
            }while(hh<=tt&&s[q[tt]][j]>=s[i][j])
            {
                tt--;
            }
            q[++tt] = i;
            s_min[i][j] = s[q[hh]][j];
        }
    }

    // 最大值
    for (int i = 0; i < n; i++)
    {
        hh = 0, tt = -1;
        for (int j = 0; j < m; j++)
        {
            while (hh <= tt && j - b + 1 > q[hh])
            {
                hh++;
            }
            while (hh <= tt && ans[i][q[tt]] <= ans[i][j])
            {
                tt--;
            }
            q[++tt] = j;
            s[i][j] = ans[i][q[hh]];
        }
    }
    for (int j = 0; j < m; j++)
    {
        hh = 0, tt = -1;
        for (int i = 0; i < n; i++)
        {
            while (hh <= tt && i - a + 1 > q[hh])
            {
                hh++;
            }
            while (hh <= tt && s[q[tt]][j] <= s[i][j])
            {
                tt--;
            }
            q[++tt] = i;
            s_max[i][j] = s[q[hh]][j];
        }
    }

    LL res = 0;

    for (int i = a - 1; i < n;i++)
    {
        for (int j = b - 1; j < m;j++)
        {
            res = (res +(LL)s_min[i][j] * s_max[i][j]) % MOD;
        }
    }

    cout << res%MOD << endl;

    return 0;
}
```
# 完全背包 计数dp
## 完全背包
## 朴素完全背包
```c++
// f[i][j] = f[i - 1][j] + f[i][j - i]
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;

const int N = 1010, MOD = 1e9 + 7;

int f[N][N];

int main()
{
    int n;
    cin >> n;
    for (int i =0; i <= n; i++)
    {
        f[i][0] = 1;//容量为零，前i个物品不选也是一种方案
    }

    for (int i = 1; i <= n; i++)
    {
        for (int j = 0; j <= n; j++)
        {
            if (j >= i)
            {
                f[i][j] = (f[i - 1][j]+f[i][j - i]) % MOD;
            }
            else{
                f[i][j] = f[i - 1][j] % MOD;
            }

        }
    }
    cout << f[n][n] << endl;

    return 0;
}
```

### 优化版

## 其他状态计算

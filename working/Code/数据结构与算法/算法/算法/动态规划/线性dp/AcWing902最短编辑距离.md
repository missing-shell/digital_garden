# 线性DP

**千万不要忘记特定初始化**

```c++
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1e3 + 10;

char ans[N], bns[N];

int f[N][N];
int main()
{
    int n, m;
    cin >> n >> ans + 1;
    cin >> m >> bns + 1;

    // 初始化边界
    for (int i = 0; i <= n; i++)
    { 
        // ans 字符串的前 i 个字符变为 bns 字符串的前 0 个字符，需要 i 步
        f[i][0] = i;
    }
    for (int j = 0; j <= m; j++)
    {
        f[0][j] = j;
    }

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
            if (ans[i] != bns[j])
            {
                f[i][j] = min(f[i][j], f[i - 1][j - 1] + 1);
            }
            else
            {
                f[i][j] = min(f[i - 1][j - 1], f[i][j]);
            }
        }
    }
    cout << f[n][m] << endl;

    return 0;
}
```
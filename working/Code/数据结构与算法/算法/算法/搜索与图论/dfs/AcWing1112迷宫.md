- 确保考虑到所有情况
- 不要使用特别复杂的嵌套`if else`语句，影响可读性

## 易错
#tips
- 确保数据及时输入
- 多次循环使用相同变量时，确保变量的每次初始化，避免被之前的数据所影响
- `return 0`的位置千万别放在循环中
- 检查`==`和`=`

```c++
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 110;
const int dx[] = {-1, 1, 0, 0}, dy[] = {0, 0, -1, 1};

bool ans[N][N];
bool st[N][N];

int n, xa, ya, xb, yb;

int dfs(int x, int y)
{
    if (ans[x][y])
    {
        return 0;
    }
    if (x == xb && y == yb)
    {
        return 1;
    }
    st[x][y] = true;
    for (int i = 0; i < 4; i++)
    {
        int a = x + dx[i], b = y + dy[i];
        if (a < 0 || a >= n || b < 0 || b >= n)
        {
            continue;
        }
        if (st[a][b])
        {
            continue;
        }
        if (dfs(a, b))
        {
            return 1;
        }
    }
    return 0;
}
int main()
{
    int k;
    cin >> k;

    while (k--)
    {
        cin >> n;
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                char str;
                cin >> str;
                if (str == '.')
                {
                    ans[i][j] = 0;
                    
                }
                else
                {
                    ans[i][j] = 1;
                }
            }
        }

        cin >> xa >> ya >> xb >> yb;

        memset(st, 0, sizeof(st));
        int res = 0;
        if (ans[xa][ya] || ans[xb][yb])
        {
            res = 0;
        }
        else
        {
            res = dfs(xa, ya);
        }

        if (res)
        {
            cout << "YES" << endl;
        }
        else
        {
            cout << "NO" << endl;
        }
    }
    return 0;
}
```
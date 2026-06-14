# `BFS` `DFS` `FloodFill`
- 思路
	求解最初有多少岛屿
	将被被淹没的地方设置为`false`
	再次求解岛屿数量

	如何求解岛屿数量

**题目一看就是“连通块问题”，是基础搜索。用DFS或BFS都行：遍历一个连通块（找到这个连通块中所有的’#‘，并标记已经搜过，不用再搜）；再遍历下一个连通块…；遍历完所有连通块，统计有多少个连通块。**
**因为每个像素点只用搜一次且必须搜一次，所以复杂度是O$(n^2)$的，不可能更好了。**
## `bfs`
- 参考：y总打卡
```c++
#include <iostream>
#include <queue>
#include <cstdio>
#include <algorithm>

using namespace std;
typedef pair<int, int> PII;

const int N = 1010;
const int dx[] = {-1, 1, 0, 0}, dy[] = {0, 0, -1, 1};

char ans[N][N];
PII q[N * N];
bool st[N][N];
int n;

void bfs(int x, int y, int &total, int &bound)
{
    int hh = 0, tt = -1;
    q[++tt] = {x, y};
    st[x][y] = true;
    while (hh <= tt)
    {
        auto t = q[hh++];
        total++;

        bool is_bound = false;

        for (int i = 0; i < 4; i++)
        {
            int a = t.first + dx[i], b = t.second + dy[i];
            if (a < 0 || a >= n || b < 0 || b >= n)//越界
            {
                continue;
            }
            if (st[a][b])//避免重复判断
            {
                continue;
            }
            if (ans[a][b] == '.')
            {
                is_bound = true;
                continue;
            }

            q[++tt] = {a, b};//陆地入队
            st[a][b] = true;
        }
        if (is_bound)
        {
            bound++;
        }
    }
}
int main()
{
    scanf("%d", &n);

    for (int i = 0; i < n; i++)
    {
        scanf("%s", ans[i]);
    }

    int res = 0;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            //找到一个岛屿，不断向外拓展
            if (!st[i][j] && ans[i][j] == '#')
            {
                int total = 0, bound = 0;//岛屿数目及边界数目
                bfs(i, j, total, bound);
                if (total == bound)
                {
                    res++;
                }
            }
        }
    }
    printf("%d\n", res);

    return 0;
}
```
## `dfs`
```c++
#include <iostream>
#include <queue>
#include <cstdio>
#include <algorithm>

using namespace std;

const int N = 1010;
const int dx[] = {-1, 1, 0, 0}, dy[] = {0, 0, -1, 1};

char ans[N][N];
bool st[N][N];
int n;

void dfs(int x, int y, int &total, int &bound)
{
    st[x][y] = true;

    total++;

    bool is_bound = false;

    for (int i = 0; i < 4; i++)
    {
        int a = x+ dx[i], b =y + dy[i];
        if (a < 0 || a >= n || b < 0 || b >= n) // 越界
        {
            continue;
        }
        if (st[a][b]) // 避免重复判断
        {
            continue;
        }
        if (ans[a][b] == '.')
        {
            is_bound = true;
            continue;
        }

        dfs(a, b, total, bound);
    }
    if (is_bound)
    {
        bound++;
    }
}
int main()
{
    scanf("%d", &n);

    for (int i = 0; i < n; i++)
    {
        scanf("%s", ans[i]);
    }

    int res = 0;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            // 找到一个岛屿，不断向外拓展
            if (!st[i][j] && ans[i][j] == '#')
            {
                int total = 0, bound = 0; // 岛屿数目及边界数目
                dfs(i, j, total, bound);
                if (total == bound)
                {
                    res++;
                }
            }
        }
    }
    printf("%d\n", res);

    return 0;
}
```
核心就是处理输入以及[[AcWing902最短编辑距离]]

`!注意事项`
- 注意题目范围，字符串长度和`n,m`长度不相关
- 多组数据，确保相关数据每次都初始化`res`,`f[N][M]`

## 处理输入

- 输入的多个字符串均从下标1开始
```c++
	for (int i = 1; i <= n; i++)
    {
        cin >> (ans[i] + 1);
    }
```

- 计算`ans[N][N]`中的字符串长度
```c++
int a=strlen(ans[k]);//错误，下标从1开始记录

int a = strlen(ans[k] + 1)
```

## code
```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;

const int N = 1010, M = 15;

char ans[N][M];

int f[N][M];
int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);
    cout.tie(0);

    int n, m;
    cin >> n >> m;

    for (int i = 1; i <= n; i++)
    {
        cin >> (ans[i] + 1);
    }

    while (m--)
    {

        int limit;
        char s[N];
        cin >> s + 1 >> limit;

        int res = 0;
        for (int k = 1; k <= n; k++)
        {

            int a = strlen(ans[k] + 1), b = strlen(s + 1);

            for (int i = 0; i <= a; i++)
            {
                f[i][0] = i;
            }
            for (int j = 0; j <= b; j++)
            {
                f[0][j] = j;
            }

            for (int i = 1; i <= a; i++)
            {
                for (int j = 1; j <= b; j++)
                {
                    f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
                    f[i][j] = min(f[i][j], f[i - 1][j - 1] + (ans[k][i] != s[j]));
                }
            }
            if (f[a][b] <= limit)
            {
                res++;
            }
        }
        cout << res << endl;
    }
    return 0;
}
```

## 优化代码结构，将每轮比较抽象为函数
```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;

const int N = 1010, M = 15;

char ans[N][M];

int f[N][M];

int edit_distance(char a[], char b[]) 
{
    int la = strlen(a + 1), lb = strlen(b + 1);

    for (int i = 0; i <= la;i++)
    {
        f[i][0] = i;
    }
    for (int j = 0; j <= lb;j++)
    {
        f[0][j] = j;
    }

    for (int i = 1; i <= la;i++)
    {
        for (int j = 1; j <= lb;j++)
        {
            f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
            f[i][j] = min(f[i][j], f[i - 1][j - 1] + (a[i] != b[j]));
        }
    }
    return f[la][lb];
}
int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);
    cout.tie(0);

    int n, m;
    cin >> n >> m;

    for (int i = 1; i <= n; i++)
    {
        cin >> (ans[i] + 1);
    }

    while (m--)
    {
        int limit;
        char s[N];
        cin >> s + 1 >> limit;

        int res = 0;
        for (int k = 1; k <= n; k++)
        {
            if (edit_distance(ans[k],s)<=limit)
            {
                res++;
            }
        }
        cout << res << endl;
    }
    return 0;
}

```
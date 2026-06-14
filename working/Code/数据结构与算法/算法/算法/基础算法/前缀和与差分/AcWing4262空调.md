# 贪心 差分
- 题意：对原序列进行加减一实现目标序列
- 时间复杂度：$O(n)$
- 目标序列与原序列的差值为`s[N]`,求出其差分序列`ans[N]`

- 实现目标的方式
	选择任意两项，一项加一，另一项减一 =>  原数组选取中间一段+1/-1
	选择任意一项，加一或减一 =>   原数组从某个位置一直+1/-1到结束，或从开始一直+1/-1到某位置。

+ 如果差分数组，负数和 == 正数和，那就全部采用规则1，操作方案数=负数和=正数和。否则，一部分采用规则1，一部分采用规则2，操作方案数=负数和或正数和大的那个。
```c++
#include <iostream>

using namespace std;

const int N = 1e5 + 10;
int ans[N], s[N], p[N], t[N];

int main()
{
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> p[i];
    }
    int l = 0, r = 0;
    for (int i = 1; i <= n; i++)
    {
        cin >> t[i];
        s[i] = p[i] - t[i];
        ans[i] = s[i] - s[i - 1]; // 差分数组
        if (ans[i] < 0)
        {
            l -= ans[i];
        }
        else
        {
            r += ans[i];
        }
    }

    cout << max(l, r) << endl;

    return 0;
}
```
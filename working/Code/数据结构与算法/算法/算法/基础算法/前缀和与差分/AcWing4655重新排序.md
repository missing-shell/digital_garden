# 贪心 差分

## Part 1：贪心

累加每个`s[i]`的被求和次数`cnt[i]`,容易贪心得到，被求和次数越多的肯定得放越大的数（可用邻项交换证明）。

我们可以先统计原来的求和的总和 `a`,再将`s[N]`和`cnt[N]`数组从小到大排好序，最后依次相乘求和得到新的总和`b`。

## Part 2：差分

统计求和次数时注意到只有修改操作而没有查询操作，于是可以用差分来维护。

具体地，我们建立一个都为 0的差分数组`cnt[N]`，

**注意：原来的总和和最大总和都要用 `long long`来存储。**

时间复杂度：\[O(n \log n)\]

```c++
#include <iostream>
#include<algorithm>

using namespace std;

const int N = 1e5 + 10;

typedef long long int LL;//应该提前根据题目设置，而不是在运行错误之后再补救

int s[N], cnt[N];

int main()
{
    int n;
    cin >> n;

    for (int i = 1; i <= n; i++)
    {
        cin >> s[i];
    }

    int m;
    cin >> m;
    LL a = 0, b = 0;

    for (int i = 1; i <= m; i++)
    {
        int l, r;
        cin >> l >> r;
        cnt[l]++;
        cnt[r + 1]--;//差分数组
    }
    for (int i = 1; i <= n;i++)
    {
        cnt[i] += cnt[i - 1];//记录每一位数据贡献的次数
        a += (LL)cnt[i] * s[i];
    }
    sort(s + 1, s + n + 1);
    sort(cnt + 1, cnt + n + 1);
    for (int i = 1; i <= n;i++)
    {
        b += (LL)cnt[i] * s[i];
    }
    cout << b - a << endl;

    return 0;
}
```

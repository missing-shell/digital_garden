## 求右边界
- 使用`l=mid`作为更新条件，mid即是当前最大值，像右逼近
- 将二维问题划分为一维
```c++
#include <stdio.h>
#include <iostream>

using namespace std;

const int N = 1e5 + 10;

int h[N], w[N];

bool check(int n, int x, int k)
{
    bool ret = false;
    int sum = 0;
    for (int i = 0; i < n; i++)
    {
        // 将二维问题分解为长和宽的一维问题
        // 先除后乘避免溢出
        sum += (h[i] / x) * (w[i] / x);
        if (sum >= k)
        {
            ret = true;
            break;
        }
    }
    return ret;
}

int main()
{
    int n, k;
    cin >> n >> k;
    for (int i = 0; i < n; i++)
    {
        cin >> h[i] >> w[i];
    }
    int l = 1, r = 1e5; // 边长的范围为1~1e5
    while (l < r)
    {
        // 二分模板
        // 当边长取mid时，能够分得k快巧克力
        // 在mid及其左边的所有值均满足条件，最大值即为mid
        // 如果当前的mid满足这个条件，那么l就有可能是我们所需要的目标值，所以用l=mid来更新
        int mid = (l + r + 1) >> 1;
        if (check(n, mid, k))
        {
            l = mid;
        }
        else
        {
            r = mid - 1;
        }
    }
    cout << l << endl;

    return 0;
}
```
## 使用左边界计算
- 求不满做条件的最小值
- 向左逼近
- 注意溢出，`LL`
- 值的取值也不再是1~1e5，而是2~1e5+1
- 所求的结果为不满足条件的最小值，最终输出结果需要减一
```c++
#include <stdio.h>
#include <iostream>

using namespace std;
typedef long long LL;
const int N = 1e5 + 10;

int h[N], w[N];

bool check(int n, int x, int k)
{
    LL sum = 0;
    for (int i = 0; i < n; i++)
    {
        // 将二维问题分解为长和宽的一维问题
        // 先除后乘避免溢出
        sum += (LL)(h[i] / x) * (w[i] / x);
    }
    return sum < k;
}

int main()
{
    int n, k;
    cin >> n >> k;
    for (int i = 0; i < n; i++)
    {
        cin >> h[i] >> w[i];
    }
    int l = 1, r = 1e5 + 1; // 边长的范围为1~1e5，所以不满足条件的最小值可能是1e5+1;
    while (l < r)
    {
        // 二分模板
        // 当边长取mid时，能够分得k快巧克力
        // 在mid及其右边的所有值均不满足条件
        // 如果当前的mid满足这个条件，那么r就有可能是我们所需要的目标值，所以用r=mid来更新
        // 向左逼近，求得不满足条件的最小值
        int mid = (l + r) >> 1;
        if (check(n, mid, k))
        {
            r = mid;
        }
        else
        {
            l = mid + 1;
        }
    }
    cout << l - 1 << endl;

    return 0;
}
```
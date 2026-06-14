- **二分是二分性而不是单调性 只要满足可以找到一个值一半满足一半不满足即可 而不用满足单调性**


①：我们要找的是 有没有一段不小于F的区间，使这段区间的平均数尽可能的大，如果我们找到了一段连续的区间且区间长度不小于F且平均数大于我们二分的平均数 那么大于这个数且区间也满足的一定满足了 我们直接判断正确即可

②：因为我们要找一段区间的平均数，根据平均数的一个基本应用，显而易见，对于一段序列，每个数减去我们所算的平均数，如果大于0 那么他本身就大于平均数，如果小于0 那么它本身就小于平均数 此时我们就能算出哪些数大于0 哪些数小于0 ，之后我们再使用前缀和，就能判断一个区间内的平均值是否大于或小于我们二分的平均数了

 
 [参考](https://www.acwing.com/solution/content/1148/)
 
```c++
#include <stdio.h>
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1e5 + 10;

int ans[N];
double sum[N];
bool check(int n, int k, double avg)
{
    for (int i = 1; i <= n; i++)
    {
        sum[i] = sum[i - 1] + ans[i] - avg;
    }

    double minv = 0;
    bool leap = false;
    // for (int j = k; j <= n; j++)  //暴力
    // {
    //     for (int i = 0;  i <= j-k; i++)
    //     {
    //         minv = min(minv, sum[i]);
    //     }
    //     if (sum[j] - minv >= 0)
    //     {
    //         leap = true;
    //         break;
    //     }
    // }
    for (int i = 0, j = k; j <= n; j++,i++)
    {
        minv = min(minv, sum[i]);
        if (sum[j] - minv >= 0)
        {
            leap = true;
            break;
        }
    }

    return leap;
}
int main()
{
    int n, k;
    cin >> n >> k;
    int sum = 0;

    double l = 0, r = 0;//实数
    for (int i = 1; i <= n; i++)
    {
        cin >> ans[i];
        r = max(r, (double)ans[i]);
    }

    while (r - l > 1e-5)
    {
        double mid = (l + r) / 2;//实数不可以用>>1
        if (check(n, k, mid))
        {
            l = mid;
        }
        else
        {
            r = mid;
        }
    }
    cout << (int)(r * 1000) << endl;//求的是极大值，所以要用右端点*1000（eg: l=4.45,r=4.50)
    return 0;
}
```
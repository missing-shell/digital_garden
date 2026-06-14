- 用数学的方式理解题目
- 以[A/V]为纵坐标，V为横坐标，建立二维坐标系，
![[acwing4956.jpg]]
```c++
#include <stdio.h>
#include <iostream>

using namespace std;
typedef long long LL;
const int N = 1e4 + 10;

int A[N], B[N];
LL k;

int mid_sort1(int n)
{
    LL l = 1, r = k; // 转化率最小为1，不能为0
    while (l < r)
    {
        LL mid = (l + r) >> 1;
        int leap = true;
        for (int i = 0; i < n; i++)
        {

            if ((A[i] / mid) > B[i])
            {
                leap = false;
                break;
            }
        }
        if (leap)
        {
            // 当前所有值都在最小值的右边
            // 向下取整，分段函数，具有单调性
            r = mid;
        }
        else
        {
            l = mid + 1;
        }
    }
    return l;
}

int mid_sort2(int n)
{
    LL l = 1, r = k;
    while (l < r)
    {
        LL mid = (l + r + 1) >> 1;
        int leap = true;
        for (int i = 0; i < n; i++)
        {
            if ((A[i] / mid) < B[i])
            {
                leap = false;
                break;
            }
        }
        if (leap)
        { 
            // 最大值大于当前所有值
            // 向右趋近
            l = mid;
        }
        else
        {
            r = mid - 1;
        }
    }
    return l;
}

int main()
{
    int n;
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        cin >> A[i] >> B[i];
        k += A[i];
    }
    int min = mid_sort1(n);
    int max = mid_sort2(n);
    cout << min << ' ' << max << endl;

    return 0;
}
```
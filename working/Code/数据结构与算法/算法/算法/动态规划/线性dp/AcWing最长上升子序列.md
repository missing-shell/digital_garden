## 二分法
### stl 版
- 思路
	如果子序列长度相同，那么末尾元素较小的更有优势，反过来用dp针对相同长度情况下最小的末尾元素求解。
	dp[i]：长度为i+1的上升子序列中末尾元素的最小值（不存在的则为INF)
```c++
#include <bits/stdc++.h>

using namespace std;
int lengthOfLIS(vector<int> &ans)
{
    // 特判空序列
    int n = ans.size();
    if (n == 0)
    {
        return 0;
    }
    vector<int> dp;

    for (int i = 0; i < n; i++)
    {   // 二分法找到第一个大于等于 ans[i] 的元素的位置
        int pos = lower_bound(dp.begin(), dp.end(), ans[i]) - dp.begin();
        // 如果没找到，就把 ans[i] 直接加入到 状态数组
        if (pos == dp.size())
        {
            dp.push_back(ans[i]);
        }
        else // 否则，用 ans[i] 替换该位置元素
        {
            dp[pos] = ans[i];
        }
    }
    // 状态数组的长度就是最长子序列的长度
    return (int)dp.size();
}

int main()
{
    int n;
    cin >> n;
    vector<int> ans;
    for (int i = 0; i < n; i++)
    {
        int x;
        cin >> x;
        ans.push_back(x);
    }
    int res = lengthOfLIS(ans);

    cout << res << endl;
    return 0;
}
```
### 手写二分
```c++
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n;
int ans[N];
int q[N];

int main()
{

    cin >> n;

    for (int i = 0; i < n; i++)
    {
        cin >> ans[i];
    }

    int len = 0;

    for (int i = 0; i < n; i++)
    {
        int l = 0, r =len;
        while (l < r)
        {
            int mid = (l + r + 1) / 2;
            if (q[mid] < ans[i])
            {
                l = mid;
            }
            else
            {
                r = mid - 1;
            }
        }
        len = max(len, r + 1);
        q[r + 1] = ans[i];
    }

    cout << len << endl;

    return 0;
}
```
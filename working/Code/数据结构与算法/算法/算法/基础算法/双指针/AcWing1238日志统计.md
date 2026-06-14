# 双指针  滑动窗口

- 题目条件“长度为D的时间段的点赞次数”可知题目需要对时刻进行遍历，使用`sort`对数组按照时刻进行排序，避免后续出现的点赞对当前点赞数的影响。
- 很显然是一对多的情况，所以需要使用`pair<int,int>`类型
- `for`循环中的时间复杂度为`d*`$O(n)$,`sort`的时间复杂为` 
$O(log n)$，所以总的时间复杂度为$O(log n)$。
## 双指针优化
```c++
#include <iostream>
#include<algorithm>
using namespace std;

typedef pair<int, int> PII;

const int N = 1e5 + 10;

PII q[N];

bool st[N];//记录id状态
int cnt[N];//记录点赞次数


int main()
{
    int n, d, k;
    cin >> n >> d >> k;

    for (int i = 0; i < n;i++)
    {
        cin >> q[i].first >> q[i].second;
    }

    sort(q, q + n);//默认以first排序，按时刻遍历（长度为D的时间段）

    for (int i = 0,j=0; i < n;i++)
    {
        int t = q[i].second;
        cnt[t]++;

        while (q[i].first - q[j].first >= d)//宽度为d的滑动窗口
        {
            cnt[q[j].second]--;//左边已经出窗口了,先删再移
            j++;//窗口右移
        }
        if(cnt[t]>=k)
        {
            st[t] = true;
        }
    }
    for (int i = 0; i < 1e5;i++)
    {
        if(st[i])
        {
            cout << i <<endl;
        }
    }

    return 0;
}
```
## 单调队列
- 不推荐这种方式，使代码更难理解，且`q[N]`的使用不是必须的
```c++
#include <iostream>
#include <algorithm>
#include <set>

using namespace std;

typedef pair<int, int> PII;

const int N = 1e5 + 10;

PII blog[N];

int q[N], hh = 0, tt = -1;//滑动窗口的左右指针

int cnt[N];

set<int> res;

int main()
{
    int n, d, k;
    cin >> n >> d >> k;

    for (int i = 0; i < n; i++)
    {
        cin >> blog[i].first >> blog[i].second;
    }

    sort(blog, blog + n); // 默认以first排序，按时刻遍历（长度为D的时间段）

    for (int i = 0; i < n; i++)
    {
        while (hh <= tt && blog[i].first - blog[q[hh]].first >= d)
        {
            cnt[blog[q[hh]].second]--;
            hh++;
        }
        cnt[blog[i].second]++;
        if (cnt[blog[i].second] >= k)
        {
            res.insert(blog[i].second);
        }
        q[++tt] = i;
    }
    for (auto i : res)
    {
        cout << i << endl;
    }

    return 0;
}
```
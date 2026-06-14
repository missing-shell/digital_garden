# 归并排序  树状数组  线段树
## 归并排序
- `pair<int,int>ans[N];`//{身高，编号}
	**参考逆序对作答时需要考虑交换导致的`i`与之前的`i`不一致的问题，所以需要使用`pair<int,int>` 
- 相对于i来说,j 前面的数都比它小, j 前面的数都和 i 交换过
	`(j-1)-(mid+1)+1`
-  相对于j来说,i 后面的数都比它大, i 后面的数都和 j 交换
	`mid-i+1`
```c++
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1e5 + 10;
typedef long long LL;
typedef pair<int, int> PII;

LL cnt[N];

PII ans[N];//{身高，编号}
PII tmp[N];

void merge_sort(int l, int r)
{
    if (l >= r)
    {
        return;
    }
    int mid = (l + r) >> 1;
    merge_sort(l, mid);
    merge_sort(mid + 1, r);

    int k = 0, i = l, j = mid + 1;
    while (i <= mid && j <= r)
    {
        if (ans[i] <= ans[j])
        {
            // 相对于i来说,j 前面的数都比它小, j 前面的数都和 i 交换过
            //(j-1)-(mid+1)+1
            cnt[ans[i].second] += j - mid - 1;
            tmp[k++] = ans[i++];
        }
        else
        { 
            // 相对于j来说,i 后面的数都比它大, i 后面的数都和 j 交换
            cnt[ans[j].second] += mid - i + 1;
            tmp[k++] = ans[j++];
        }
    }
    while (i <= mid)
    {
        cnt[ans[i].second] += j - mid - 1;
        tmp[k++] = ans[i++];
    }
    while (j <= r)
    {
        tmp[k++] = ans[j++];
    }
    for (int i = l, j = 0; i <= r; i++, j++)
    {
        ans[i] = tmp[j];
    }
}
int main()
{
    int n;
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        cin >> ans[i].first;
        ans[i].second = i;
    }
    merge_sort(0, n - 1);
    LL res = 0;
    for (int i = 0; i < n; i++)
    {
        res += cnt[i] * (cnt[i] + 1) / 2;
    }
    cout << res << endl;

    return 0;
}
```
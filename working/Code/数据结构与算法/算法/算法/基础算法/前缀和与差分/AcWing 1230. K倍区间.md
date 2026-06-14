## 错误做 法

- 缺少对前缀和数组的提前处理
- 暴力导致超时
- 没有考虑结果超过int的情况，应该使用`long long cnt`

```c++
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &ans[i]);
        sum[i] = sum[i - 1] + ans[i];
        if (ans[i] % k == 0)
        {
            cnt++;
        }
        if((sum[i]%k)==0)
        {
            cnt++;
        }
    }

    // 暴力，会超时
    for (int i = 2; i <= n; i++)
    {
        for (int j = 2; j < i; j++)
        {
            if ((sum[i] - sum[j - 1]) % k == 0)
            {
                cnt++;
            }
            if ((sum[i] - sum[j - 1]) < k)
            {
                break;
            }
        }
    }
```

## 优化

- 参考：[wuog题解](https://www.acwing.com/solution/content/6909/)

### 模运算

- `(a+b) % p = (a%p + b %p) %p`
- `(a%p + b) % p = (a%p%p + b %p) %p = (a+b) %p`
- 同理可得`(a+b+c) %p = ((a%p + b) %p + c) %p`

### 分析

题目求ans区间[l,r]的和满足K的倍数的情况，即求前缀和`(sum[r]-sum[l-1])%k==0`等价于`sum[l-1]%k==sum[r]%k`。

求出满足`sum[r]%k==t`的个数，再将其中任意两个数进行组合，特别的是，`sum[r]%k==0`本身就满足题意，所以需要提前设置`res[0]=1`。

注意结果范围，`int`会 溢出。

```c++
#include <iostream>

using namespace std;

typedef long long LL; //结果可能超出int范围

const int N = 1e5 + 10;

int ans[N], sum[N];
LL cnt[N];

int main()
{
    int n, k;
    scanf("%d%d", &n, &k);

    cnt[0] = 1;//需要初始化的变量应放在运算之前。

    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &ans[i]);
        sum[i] = (sum[i - 1] + ans[i]) % k;//前缀和优化
        cnt[sum[i]]++;
    }

    LL res = 0;

    for (int i = 0; i < k; i++)//取模之后，余数必定在0~k-1之间
    {
        res += (LL)cnt[i] * (cnt[i] - 1) / 2;//组合公式,从n种情况任意选择两种组合
    }

    printf("%ld\n", res);

    return 0;
}

```

### 进一步简化，减少轮询

```c++
#include <iostream>

using namespace std;

typedef long long LL; //结果可能超出int范围

const int N = 1e5 + 10;

int ans[N], sum[N];
LL cnt[N];

int main()
{
    int n, k;
    scanf("%d%d", &n, &k);

    cnt[0] = 1;//sum[i]%k==0，不需要和sum[j]组合也满足题意
    LL res = 0;

    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &ans[i]);
        sum[i] = (sum[i - 1] + ans[i]) % k;//前缀和优化

        res += cnt[sum[i]];
       
        cnt[sum[i]]++;
    }

    printf("%ld\n", res);

    return 0;
}

```

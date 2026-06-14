## 定义
#lowbit #树状数组

> 返回数字**二进制**最右边的`1`

- `10——1010，lowbit(10)=10;`
- 常用于求一个十进制数对应二进制数中`1`的个数
- 使用`lowbit`操作，每次`lowbit`操作截取一个数字最后一个`1`后面的所有位，每次减去`lowbit`得到的数字，直到数字减到`0`，就得到了最终`1`的个数
### `lowbit`原理
- 正数的补码等于原码，负数的补码等于反码加一

- 如一个数字原码是`10001000`，他的负数表示是补码，就是反码`+1`，反码是`01110111`，加一则是`01111000`，二者按位与得到了`1000`，就是我们想要的`lowbit`操作
## 例：求解数字二进制中1的个数
###  Lowbit解法
- `lowbit` $O(nlogn)$
```c++
#include<iostream>
using namespace std;
int lowbit(int x){
    return x&(-x);
}
int main(){
    int n;
    cin>>n;
    while(n--){
        int x;
        cin>>x;

        int res=0;
        while(x) x-=lowbit(x),res++;

        cout<<res<<' ';
    }
return 0;
}```


###  暴力
- $O(nlgn)$ 

- 对于每个数字`a`，`a&1`得到了该数字的最后一位，之后将`a`右移一位，直到位`0`，就得到了1的个数

```c++
#include<iostream>
using namespace std;
int n;
int a,k;
int main(){
    scanf("%d",&n);
    for(int i=0;i<n;i++){
        scanf("%d",&a);
        k=0;
        while(a){
            k+=a&1;
            a=a>>1;
        }
        printf("%d ",k);
    }
    return 0;
}
```

### 法三：lowbit
#lowbit 
- [[lowbit]]
#### Code
```C++
class Solution
{
public:
    int hammingWeight(int n)
    {
        int res = 0;
        while (n != 0)
        {
            res++;
            n -= (n & (-n));
        }
        return res;
    }
};
```

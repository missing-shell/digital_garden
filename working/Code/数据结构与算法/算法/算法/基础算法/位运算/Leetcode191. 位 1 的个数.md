## 概述
#位运算 #lowbit
- [191. 位1的个数](https://leetcode.cn/problems/number-of-1-bits/)
## 解答
### 法一：循环检查二进制位
- 检查第`n`位时，让`n`与$2^n$进行与运算
- 时间复杂度：$O(k)$,$k$是`int`型的二进制位数
- 空间复杂度：$O(1)$
#### Code
```c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int ret = 0;
        for (int i = 0; i < 32; i++) {
            if (n & (1 << i)) {
                ret++;
            }
        }
        return ret;
    }
};
```
### 法二：`n&(n-1)`
- *(n−1)* ： 二进制数字 `n` 最右边的 `1` 变成 `0` ，此 `1` 右边的 `0` 都变成 `1` 。
- *n&(n−1)*： 二进制数字 `n` 最右边的 `1` 变成 `0` ，其余不变。
- 时间复杂度 $O(M)$ ： `n&(n−1)` 操作仅有减法和与运算，占用 `O(1)` ；设 `M` 为二进制数字 `n` 中 `1` 的个数，则需循环 `M` 次（每轮消去一个 `1` ），占用 $O(M)$ 。
- 空间复杂度 $O(1)$ ： 变量 `res` 使用常数大小额外空间。
#### Code
```C++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int res = 0;
        while (n != 0) {
            res++;
            n &= n - 1;
        }
        return res;
    }
};
```
 
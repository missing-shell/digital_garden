### 思路
* `print_binary`函数使用递归来打印一个整数的二进制表示。它首先检查如果右移一位后的数是否大于0，如果是，那么它递归调用自己。然后，它使用`putc`函数来打印出最右边的位（0或1）。

### code
```c++
#include <stdio.h>

void print_binary(unsigned int number) {
    if (number >> 1) {
        print_binary(number >> 1);
    }
    putc((number & 1) ? '1' : '0', stdout);
}

int main() {
    unsigned int number = 10;
    printf("The binary representation of %d is: ", number);
    print_binary(number);
    printf("\n");
    return 0;
}
```
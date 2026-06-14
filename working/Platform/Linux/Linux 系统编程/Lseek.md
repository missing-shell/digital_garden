`lseek` 是一个在 Unix 和类 Unix 系统中用于**设置文件描述符**指向的文件位置的函数。它允许程序在文件中向前或向后移动文件位置指示器（即文件指针），而不需要实际读取或写入文件。

函数原型如下：

```c
#include <unistd.h>

off_t lseek(int fildes, off_t offset, int whence);
```

参数说明：
- `fildes`：文件描述符，它是通过打开文件（如使用 `open` 函数）获得的。
- `offset`：要移动的字节数。具体移动方式取决于 `whence` 参数。
- `whence`：指定 `offset` 参数之前的位置。它可以是以下值之一：
- `SEEK_SET`：文件开始（此时 `offset` 是相对于文件开始的字节偏移）。
- `SEEK_CUR`：当前文件位置（此时 `offset` 是相对于当前文件位置的字节偏移，可以是正数也可以是负数）。
- `SEEK_END`：文件结束（此时 `offset` 是相对于文件结束的字节偏移，通常用于移动到文件末尾之后的位置）。

返回值：
- 成功时，`lseek` 返回新的文件位置偏移，即文件位置指示器的当前位置，从文件开始计算的字节数。
- 出错时，返回 `(off_t)-1`，并设置 `errno` 以指示错误类型。

特点：
- `lseek` 仅改变文件位置指示器，不进行实际的读写操作。
- 对于普通文件，`lseek` 可以用来随机访问文件的任意位置。
- 对于某些特殊文件（如管道和终端设备），`lseek` 可能返回错误，因为这些文件不支持随机访问。

使用 `lseek` 的一个简单示例：

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    off_t newpos = lseek(fd, 5, SEEK_SET); // 将文件指针移动到文件开始的第5个字节
    if (newpos == (off_t)-1) {
        perror("lseek");
        close(fd);
        return 1;
    }

    // 可以继续读取文件，从新的文件位置开始
    // ...

    close(fd); // 关闭文件
    return 0;
}
```

在上面的代码中，我们首先打开一个文件，然后使用 `lseek` 将文件指针移动到文件开始的第5个字节。之后，我们可以从这个新的位置开始读取文件。如果在 `lseek` 调用中遇到错误，我们会打印出错误信息并退出程序。最后，我们关闭了文件。
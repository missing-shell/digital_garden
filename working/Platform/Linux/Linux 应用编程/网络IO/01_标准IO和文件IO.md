## 定义
### 文件IO
- 文件IO就是直接的调用内核提供的系统调用函数（用户和内核的直接双向调用）
- 头文件是`unistd.h`，依赖于*操作系统*，围绕**文件**操作
### 标准IO
- 标准IO就是间接调用系统调用函数，是C库函数
- 头文件是`stdio.h`，移植性好，围绕**流**操作

## 文件IO
### 文件描述符
- 文件描述符的本质是一个非负整数，当我们打开一个文件时，系统会给我们分配一个文件描述符
- 当我们使用`open`函数返回的这个文件描述符会表示该文件，并将其作为参数传递给`read`或`write`函数
- 在`posix.1`应用程序里面，文件描述符`0,1,2`分别对应于**标准输入、标准输出和标准出错**

### `open()`
- 头文件
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

/**
 * \brief 打开文件并获取文件描述符。
 *
 * 该函数尝试打开指定路径名 `pathname` 所指向的文件，并根据 `flags` 参数指定的选项来访问文件。
 * 如果需要，还可以设置文件的访问权限 `mode`。
 *
 * \param pathname 要打开的文件的路径名。
 * \param flags 文件打开的选项，可以是以下值的组合：
 *   - O_RDONLY：以只读方式打开文件。
 *   - O_WRONLY：以只写方式打开文件。
 *   - O_RDWR：以读写方式打开文件。
 *   - O_CREAT：如果文件不存在，创建文件。
 *   - O_TRUNC：如果文件存在，截断文件内容到0长度。
 *   - O_APPEND：每次写操作都从文件末尾开始。
 *   - 其他选项，具体取决于系统实现。
 * \param mode 如果创建新文件，这是新文件的访问权限。具体值取决于系统。
 *
 * \return 成功时返回新的文件描述符；失败时返回-1，并设置全局变量 `errno` 以指示错误。
 *
 * \note 打开文件时，应检查返回的文件描述符，并在不再需要时使用 `close` 函数关闭它。
 * \note `flags` 参数的组合和行为可能因系统而异，某些选项可能不被所有系统支持。
 * \note 使用 `O_CREAT` 选项时，必须同时指定 `mode` 参数。
 * \see close, read, write, lseek
 */
int open(const char *pathname, int flags, mode_t mode);
```
- 示例：open.c
```c
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int main(int argc,char *argv[])
{
        int fd;
        fd=open("a.c",O_CREAT|O_RDWR,0666);
        if(fd<0)
        {
                printf("open is error %d\n",fd);
        }
        printf("fd is %d\n",fd);
        return 0;
}
```
- 查看文件权限
```shell
imx@ubuntu:~/WorkSpace/02$ ll a.c
-rw-rw-r-- 1 imx imx 0 Aug  1 01:47 a.c
#rw:6
#rw:6
#r:4
```
- 在开发板查看权限使用`ls -al`
```
root@okmx8mm:/mnt/nfs# ls -al
total 48
drwxr-xr-x 2 root root  4096 Aug  1 09:01 .
drwxr-xr-x 4 root root  4096 Jul 31 09:13 ..
-rw-r--r-- 1 root root     0 Aug  1 09:01 a.c
-rwxr-xr-x 1 root root 18192 Aug  1 09:01 open
-rwxr-xr-x 1 root root 18152 Aug  1 08:13 test
```
- 查看掩码
```shell
imx@ubuntu:~/WorkSpace/02$ umask
0002
```
- 掩码计算
```shell
mode_t&~(umask)
# ~(0002)=07775
# 0666&07775=664
# 结果同`ll a.c`
```

### `close()`
- 文件描述符有限，打开后需及时关闭
- 头文件
```c
 #include <unistd.h>
 
/**
 * \brief 关闭一个文件描述符。
 *
 * 该函数关闭指定的文件描述符 `fd`，释放与该描述符关联的任何系统资源。
 *
 * \param fd 要关闭的文件描述符。
 *
 * \return 成功时返回0，失败时返回-1，并设置全局变量 `errno` 以指示错误。
 *
 * \note 一旦文件描述符被关闭，任何对该描述符的进一步操作都将导致未定义的行为。
 * \note 即使关闭文件描述符时发生错误，该描述符也不再可用，因此不应再次关闭它。
 * \note 某些系统可能允许文件描述符被重新打开，但这并不是可移植的做法。
 * \see open, read, write
 */
int close(int fd);
```

### `read()`
- 头文件
```c
#include <unistd.h>

/**
 * \brief 从文件描述符读取数据。
 *
 * 该函数尝试从文件描述符 `fd` 指向的文件或其他对象中读取最多 `count` 个字节的数据到缓冲区 `buf`。
 *
 * \param fd 要读取数据的文件描述符。
 * \param buf 指向存储读取数据的缓冲区的指针。
 * \param count 要读取的最大字节数。
 *
 * \return 成功时返回实际读取的字节数。如果返回的字节数小于 `count`，则表示可能已经到达文件末尾或发生了其他情况。
 *         如果发生错误，则返回 -1，并且会设置全局变量 `errno` 来指示错误类型。
 *
 * \note 如果 `fd` 指向的是管道或套接字，则 `read` 函数的行为可能会有所不同。
 * \note `read` 函数不会修改 `buf` 指向的缓冲区中未实际读取的数据部分。
 * \note 对于非阻塞文件描述符，如果没有任何数据可读，`read` 将立即返回0。
 * \see write, open, close, lseek
 */
ssize_t read(int fd, void *buf, size_t count);
```
- 返回实际读到的字节数

### `write()`
- 头文件
```c
#include <unistd.h>

/**
 * \brief 向文件描述符写入数据。
 *
 * 该函数将 `count` 字节的数据从缓冲区 `buf` 写入到文件描述符 `fd` 指向的文件或设备。
 *
 * \param fd 目标文件描述符，必须是一个有效的文件描述符。
 * \param buf 指向要写入数据的缓冲区的指针。
 * \param count 要写入的字节数。
 *
 * \return 成功时返回实际写入的字节数。如果写入的字节数小于 `count`，则可能发生了错误或文件结束。
 *         失败时返回 -1，并设置errno以指示错误。
 *
 * \note 写入操作可能不会立即完成。如果写入的数据量很大，系统可能会将数据分批写入。
 * \note 对于某些文件描述符（如管道），写入的数据量可能受到限制。
 * \see read, open, close
 */
ssize_t write(int fd, const void *buf, size_t count);
```

### `lseek()`
- 头文件
```c
#include <sys/types.h>
#include <unistd.h>

/**
 * \brief 改变文件描述符的文件偏移量。
 *
 * 该函数设置文件描述符 `fd` 的偏移量，使其指向文件的指定位置。
 * 偏移量 `offset` 可以是正数、负数或零，取决于 `whence` 参数。
 *
 * \param fd 需要改变偏移量的文件描述符。
 * \param offset 要移动的字节数。正数表示向前移动，负数表示向后移动。
 * \param whence 偏移量的参考点，可以是以下之一：
 *   - SEEK_SET：文件的开头。
 *   - SEEK_CUR：当前文件位置。
 *   - SEEK_END：文件的结尾。
 *
 * \return 成功时返回新的文件偏移量；失败时返回 (off_t)-1，并设置errno以指示错误。
 *
 * \note 此函数不会改变文件的实际内容，只是改变文件描述符的读取/写入位置。
 * \see open, read, write
 */
off_t lseek(int fd, off_t offset, int whence);
```
## 标准IO
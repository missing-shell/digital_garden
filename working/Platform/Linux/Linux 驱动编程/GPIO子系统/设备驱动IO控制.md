操作`GPIO`：寄存器-->`GPIO`函数
`GPIO`逻辑判断：应用层-->`write`-->`copy_from_user`-->`kbuf[0]`
## 设备驱动IO控制原理
- 在内核`3.0` 以前，`ioctl`接口的名字叫`ioctl`;内核`3.0`以后，`ioctl`接口的名字叫`unlocked_ioctl`。
- `unlocked_ioctl`就是`ioctl`接口，功能和对应的系统调用均没有发生变化。
### unlocked_ioctl
> `unlocked_ioctl`和`read/write`函数有什么异同呢？
- *相同点*：都可以往内核写数据。
- *不同点*：`read`函数只能完成读的功能，`write`只能完成写的功能。读取大数据的时候**效率高**。`ioctl`既可以读也可以写，读取大数据的时候效率不高。

> `ioctl`函数的任务是对工作参数进行**设置和查询**，`write`和`read`函数专注于**数据传输**。
### cmd
在驱动程序里，`ioctl()`函数上传送的变量`cmd`是应用程序用于区别设备驱动程序请求处理内容的值。

- `cmd`除了可*区别数字*外，还包含有助于处理的几种相应信息。`cmd`的大小为`32`位，共分`4`个域。
- *0-7*：命令的**编号**，范围是`0-255`。
- *8-15*：命令的~~幻数~~。
- *16-29*：表示传递的*数据大小*。
- *30-31*：代表读写的*方向*。
- *00*：表示用户程序和驱动程序无**数据传递**
- *10*：表示用户程序从驱动里面读数据
- *01*：表示用户程序向驱动里面写数据
- *11*：先写数据到驱动里面然后在从驱动里面把数据读出来。
#### 合成宏
- *_IO(type,nr)* ：没有数据传递的命令
- *_IOR(type,nr,size)* ：定义从驱动中读取数据的命令
- *_IOW(type,nr,size)* ：定义向驱动写入数据的命令
- *_IOWR(type,nr,size)* ：定义数据交换类型的命令，先写入数据，再读取数据这类命令。

> 参数说明：
- *type*：表示命令组成的模式，也就是`8~15`位
- *nr*：表示命令组成的编号，也就是`0~7`位
- *size*：表示命令组成的参数传递大小，即传递的==数据类型==，如要传递`4`字节，就可以写成`int`。
#### 分解宏

- *_IOC_DIR(nr)* ：分解命令的方向，`30~31`位的值
- *_IOC_TYPE(nr)* ：分解命令的魔数，`8~15`位的值
- *_IOC_NR(nr)* ：分解命令的编号，`0~7`位的值
- *_IOC_SIZE(nr)* ：分解命令的复制数据大小，`16~29`位的值

> 参数说明：
- `nr` ：要分解的命令

#### 验证程序
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>
#define CMD_TEST0 _IO('L', 0)
#define CMD_TEST1 _IO('A', 1)
#define CMD_TEST2 _IOW('L', 2, int)
#define CMD_TEST3 _IOR('L', 3, int)
 
int main(int argc, char *argv[])
{
    printf("30-31 is %d \n", _IOC_DIR(CMD_TEST0));
    printf("30-31 is %d \n", _IOC_DIR(CMD_TEST3));
 
    printf("7-15 is %c \n", _IOC_TYPE(CMD_TEST0));
    printf("7-15 is %c \n", _IOC_TYPE(CMD_TEST1));
 
    printf("0-7 is %d \n", _IOC_NR(CMD_TEST2));
}
```

- 法一：使用*分解宏*将它分解
- 法二：搭配`switch`使用
## 编写测试
### 不传递数据
```c
//初始化头文件
#include <linux/init.h>
//最基本的文件，支持动态添加和卸载模块。
#include <linux/module.h>
//包含了miscdevice结构的定义及相关的操作函数。
#include <linux/miscdevice.h>
//文件系统头文件，定义文件表结构（file,buffer_head,m_inode等）
#include <linux/fs.h>
//包含了copy_to_user、copy_from_user等内核访问用户进程内存地址的函数定义。
#include <linux/uaccess.h>
//包含了ioremap、iowrite等内核访问IO内存等函数的定义。
#include <linux/io.h>
//驱动要写入内核，与内核相关的头文件
#include <linux/kernel.h>

#define CMD_TEST1 _IO('A', 1)
#define CMD_TEST0 _IO('L', 0)

ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    printk("misc_read\n ");
    return 0;
}
ssize_t misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
{
    /*应用程序传入数据到内核空间，然后控制led的逻辑，在此添加*/
    // kbuf保存的是从应用层读取到的数据
    char kbuf[64] = {0};
    // copy_from_user 从应用层传递数据给内核层
    if (copy_from_user(kbuf, ubuf, size) != 0)
    {
        // copy_from_user 传递失败打印
        printk("copy_from_user error \n ");
        return -1;
    }
    //打印传递进内核的数据
    printk("kbuf is %d\n ", kbuf[0]);
    return 0;
}
/****************************************************************************************
 - @brief misc_release : 用户空间关闭设备节点时执行此函数
 - @param inode : 文件索引
 - @param file : 文件
 - @return ：成功返回 0
 ****************************************************************************************/
int misc_release(struct inode *inode, struct file *file)
{
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
/****************************************************************************************
 - @brief misc_open : 用户空间打开设备节点时执行此函数
 - @param inode : 文件索引
 - @param file : 文件
 - @return : 成功返回 0
 ****************************************************************************************/
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
/****************************************************************************************
 - @brief misc_ioctl : 用户空间使用ioctl(int fd, unsigned long request, ...(void* arg))时
 -                      自动执行此函数，根据命令执行对应的操作
 - @param file : 设备文件
 - @param cmd : 用户空间ioctl接口命令request
 - @param value : 用户空间的arg指针，依赖于接口命令request
 - @return : 成功返回 0
 ****************************************************************************************/
long misc_ioctl(struct file *file, unsigned int cmd, unsigned long value)
{
    printk("LED ON \n");
    switch (cmd) //根据命令进行对应的操作
    {
    case CMD_TEST0:
        printk("LED ON \n");
        break;
    case CMD_TEST1:
        printk("LED OFF \n");
        break;
    }
    return 0;
}
struct file_operations misc_fops = {
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = misc_write,
    .unlocked_ioctl = misc_ioctl /* 64 bit system special */};
struct miscdevice misc_dev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "hello_misc",
    .fops = &misc_fops,
};
static int misc_init(void)
{
    int ret;
    ret = misc_register(&misc_dev);
    if (ret < 0)
    {
        printk("misc register is error \n");
    }
    printk("misc register is succeed \n");

    return 0;
}
static void misc_exit(void)
{
    misc_deregister(&misc_dev);

    printk(" misc gooodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```

- 应用程序`ioctrl.c `
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>

#define CMD_TEST0 _IO('L',0)
#define CMD_TEST1 _IO('A',1)
//#define CMD_TEST2 _IOW('L',2,int)
//#define CMD_TEST3 _IOR('L',3,int)

int main(int argc,char *argv[])
{
    int fd;
    char buf[64] ={0};
    fd = open("/dev/hello_misc",O_RDWR);
    if(fd < 0)
    {
    perror("open error \n");
    return fd;
    }
    while(1)
    {
    ioctl(fd,CMD_TEST0);
    sleep(2);
    ioctl(fd,CMD_TEST1);
    }

    return 0;
}
```
进入共享目录，加载驱动模块以后，再运行应用程序

### 写数据
```c
//初始化头文件
#include <linux/init.h>
//最基本的文件，支持动态添加和卸载模块。
#include <linux/module.h>
//包含了miscdevice结构的定义及相关的操作函数。
#include <linux/miscdevice.h>
//文件系统头文件，定义文件表结构（file,buffer_head,m_inode等）
#include <linux/fs.h>
//包含了copy_to_user、copy_from_user等内核访问用户进程内存地址的函数定义。
#include <linux/uaccess.h>
//包含了ioremap、iowrite等内核访问IO内存等函数的定义。
#include <linux/io.h>
//驱动要写入内核，与内核相关的头文件
#include <linux/kernel.h>

#define CMD_TEST1 _IO('A', 1)
#define CMD_TEST0 _IO('L', 0)
#define CMD_TEST2 _IOW('L', 2, int)
#define CMD_TEST3 _IOW('L', 3, int)

ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    printk("misc_read\n ");
    return 0;
}
ssize_t misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
{
    /*应用程序传入数据到内核空间，然后控制led的逻辑，在此添加*/
    // kbuf保存的是从应用层读取到的数据
    char kbuf[64] = {0};
    // copy_from_user 从应用层传递数据给内核层
    if (copy_from_user(kbuf, ubuf, size) != 0)
    {
        // copy_from_user 传递失败打印
        printk("copy_from_user error \n ");
        return -1;
    }
    //打印传递进内核的数据
    printk("kbuf is %d\n ", kbuf[0]);
    return 0;
}
/****************************************************************************************
 - @brief misc_release : 用户空间关闭设备节点时执行此函数
 - @param inode : 文件索引
 - @param file : 文件
 - @return ：成功返回 0
 ****************************************************************************************/
int misc_release(struct inode *inode, struct file *file)
{
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
/****************************************************************************************
 - @brief misc_open : 用户空间打开设备节点时执行此函数
 - @param inode : 文件索引
 - @param file : 文件
 - @return : 成功返回 0
 ****************************************************************************************/
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
/****************************************************************************************
 - @brief misc_ioctl : 用户空间使用ioctl(int fd, unsigned long request, ...(void* arg))时
 -                      自动执行此函数，根据命令执行对应的操作
 - @param file : 设备文件
 - @param cmd : 用户空间ioctl接口命令request
 - @param value : 用户空间的arg指针，依赖于接口命令request
 - @return : 成功返回 0
 ****************************************************************************************/
long misc_ioctl(struct file *file, unsigned int cmd, unsigned long value)
{

    switch (cmd)//根据命令进行对应的操作
    {
    case CMD_TEST2:
        printk("LED ON \n");
        printk("value is %ld\n", value);
        break;
    case CMD_TEST3:
        printk("LED OFF \n");
        printk("value is %ld\n", value);
        break;
    }
    return 0;
}
struct file_operations misc_fops = {
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = misc_write,
    .unlocked_ioctl = misc_ioctl };
struct miscdevice misc_dev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "hello_misc",
    .fops = &misc_fops,
};
static int misc_init(void)
{
    int ret;
    ret = misc_register(&misc_dev);
    if (ret < 0)
    {
        printk("misc register is error \n");
    }
    printk("misc register is succeed \n");

    return 0;
}
static void misc_exit(void)
{
    misc_deregister(&misc_dev);

    printk(" misc goodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```

- 应用程序代码`ioctrl.c`
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>

#define CMD_TEST0 _IO('L',0)
#define CMD_TEST1 _IO('A',1)

#define CMD_TEST2 _IOW('L',2,int)
#define CMD_TEST3 _IOW('L',3,int)
//#define CMD_TEST3 _IOR('L',3,int)

int main(int argc,char *argv[])
{
    int fd;
    char buf[64] ={0};
    fd = open("/dev/hello_misc",O_RDWR);
    if(fd < 0)
    {
    perror("open error \n");
    return fd;
    }
    while(1)
    {
    ioctl(fd,CMD_TEST2,0);
    sleep(2);
    ioctl(fd,CMD_TEST3,1);
    }

    return 0;
}
```
进入共享目录，卸载原来加载的驱动模块，然后再加载新的驱动模块以后，再运行应用程序

### 读数据
```c
//初始化头文件
#include <linux/init.h>
//最基本的文件，支持动态添加和卸载模块。
#include <linux/module.h>
//包含了miscdevice结构的定义及相关的操作函数。
#include <linux/miscdevice.h>
//文件系统头文件，定义文件表结构（file,buffer_head,m_inode等）
#include <linux/fs.h>
//包含了copy_to_user、copy_from_user等内核访问用户进程内存地址的函数定义。
#include <linux/uaccess.h>
//包含了ioremap、iowrite等内核访问IO内存等函数的定义。
#include <linux/io.h>
//驱动要写入内核，与内核相关的头文件
#include <linux/kernel.h>

#define CMD_TEST1 _IO('A', 1)
#define CMD_TEST0 _IO('L', 0)
#define CMD_TEST2 _IOW('L', 2, int)
#define CMD_TEST3 _IOW('L', 3, int)
#define CMD_TEST4 _IOR('L', 4, int)

ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    printk("misc_read\n ");
    return 0;
}
ssize_t misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
{
    /*应用程序传入数据到内核空间，然后控制led的逻辑，在此添加*/
    // kbuf保存的是从应用层读取到的数据
    char kbuf[64] = {0};
    // copy_from_user 从应用层传递数据给内核层
    if (copy_from_user(kbuf, ubuf, size) != 0)
    {
        // copy_from_user 传递失败打印
        printk("copy_from_user error \n ");
        return -1;
    }
    //打印传递进内核的数据
    printk("kbuf is %d\n ", kbuf[0]);
    return 0;
}
/****************************************************************************************
 - @brief misc_release : 用户空间关闭设备节点时执行此函数
 - @param inode : 文件索引
 - @param file : 文件
 - @return ：成功返回 0
 ****************************************************************************************/
int misc_release(struct inode *inode, struct file *file)
{
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
/****************************************************************************************
 - @brief misc_open : 用户空间打开设备节点时执行此函数
 - @param inode : 文件索引
 - @param file : 文件
 - @return : 成功返回 0
 ****************************************************************************************/
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
/****************************************************************************************
 - @brief misc_ioctl : 用户空间使用ioctl(int fd, unsigned long request, ...(void* arg))时
 -                      自动执行此函数，根据命令执行对应的操作
 - @param file : 设备文件
 - @param cmd : 用户空间ioctl接口命令request
 - @param value : 用户空间的arg指针，依赖于接口命令request
 - @return : 成功返回 0
 ****************************************************************************************/
long misc_ioctl(struct file *file, unsigned int cmd, unsigned long value)
{
    int val;
    switch (cmd)//根据命令进行对应的操作
    {
    case CMD_TEST2:
        printk("LED ON \n");
        printk("value is %ld\n", value);
        break;
    case CMD_TEST3:
        printk("LED OFF \n");
        printk("value is %ld\n", value);
        break;
    case CMD_TEST4:
        val = 12;
        if (copy_to_user((int *)value, &val, sizeof(val)) != 0)
        {
            printk("cpoy_to_usr error \n");
            return -1;
        }
        break;
    }
    return 0;
}
struct file_operations misc_fops = {
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = misc_write,
    .unlocked_ioctl = misc_ioctl };
struct miscdevice misc_dev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "hello_misc",
    .fops = &misc_fops,
};
static int misc_init(void)
{
    int ret;
    ret = misc_register(&misc_dev);
    if (ret < 0)
    {
        printk("misc register is error \n");
    }
    printk("misc register is succeed \n");

    return 0;
}
static void misc_exit(void)
{
    misc_deregister(&misc_dev);

    printk(" misc gooodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```

- 应用程序代码`ioctl.c `
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>

#define CMD_TEST0 _IO('L', 0)
#define CMD_TEST1 _IO('A', 1)

#define CMD_TEST2 _IOW('L', 2, int)
#define CMD_TEST3 _IOW('L', 3, int)
#define CMD_TEST4 _IOR('L', 4, int)

int main(int argc, char *argv[])
{
    int fd;
    int value;
    char buf[64] = {0};
    fd = open("/dev/hello_misc", O_RDWR);
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    while (1)
    {
        ioctl(fd, CMD_TEST4, &value);
        printf("value is %d \n", value);
        sleep(2);
    }
    return 0;
}
```
编译完成如我们进入共享目录,卸载原来加载的驱动模块，然后再加载新的驱动模块以后，再运行应用程序
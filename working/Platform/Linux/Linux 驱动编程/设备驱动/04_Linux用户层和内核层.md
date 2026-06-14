### 定义
> 驱动文件最终通过与文件操作相关的系统调用或者`C`库函数（本质也是系统调用）被访问

- 字符设备和块设备都良好地体现了“一切都是文件”的设计思想
- 掌握设备文件的读写操作，如何在`Linux`用户层和内核层之间传递数据就显得尤为重要。
- [[03_MISC驱动#设备节点（设备文件）|设备文件]]
- [[03_MISC驱动#主次设备号|设备号]]
- [[03_MISC驱动#file_opreation|文件操作集]]
### 编写驱动和应用程序
#### 驱动编写
- 在[[03_MISC驱动#编写驱动|驱动代码]]的基础上进行拓展
- 填充`file_operation`结构体
```c
//文件操作集
struct file_operations misc_fops={
 
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = misc_write,
};
```
- 编写接口函数
```c
ssize_t misc_read (struct file *file, char __user *ubuf, size_t size, loff_t *loff_t) 
{
    printk("misc_read\n ");
    return 0;
}
ssize_t misc_write (struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
 
{
   printk("misc_write\n ");
    return 0;
}
int misc_release(struct inode *inode,struct file *file){  
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
int misc_open(struct inode *inode,struct file *file){
    printk("hello misc_open\n ");
    return 0;
}
```

- 完整代码
```c
/*
 * @Descripttion: 在上一章节实现了最简单杂项设备的编写，本代码再其基础上验证内核层与应用层数据交互
 */
 
#include <linux/init.h>      //初始化头文件
#include <linux/module.h>    //最基本的文件，支持动态添加和卸载模块。
#include <linux/miscdevice.h>/*注册杂项设备头文件*/
#include <linux/uaccess.h>
#include <linux/fs.h>
 
/**
 * @name: misc_read
 * @test: 从设备中读取数据，当用户层调用函数read时，对应的，内核驱动就会调用这个函数。
 * @msg: 
 * @param {structfile} *file file结构体
 * @param {char__user} *ubuf 这是对应用户层的read函数的第二个参数void *buf
 * @param {size_t} size 对应应用层的read函数的第三个参数
 * @param {loff_t} *loff_t 这是用于存放文件的偏移量的，回想一下系统编程时，读写文件的操作都会使偏移量往后移。
 * @return {*} 当返回正数时，内核会把值传给应用程序的返回值。一般的，调用成功会返回成功读取的字节数。
如果返回负数，内核就会认为这是错误，应用程序返回-1
 */
 
ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    printk("misc_read\n ");
    return 0;
}
 
/**
 * @name: misc_write
 * @test: 往设备写入数据，当用户层调用函数write时，对应的，内核驱动就会调用这个函数。
 * @msg: 
 * @param {structfile} * filefile结构体
 * @param {constchar__user} *ubuf 这是对应用户层的write函数的第二个参数const void *buf
 * @param {size_t} size 对应用户层的write函数的第三个参数count。
 * @param {loff_t} *loff_t 这是用于存放文件的偏移量的，回想一下系统编程时，读写文件的操作都会使偏移量往后移。
 * @return {*} 当返回正数时，内核会把值传给应用程序的返回值。一般的，调用成功会返回成功读取的字节数。
 如果返回负数，内核就会认为这是错误，应用程序返回-1。
 */
 
ssize_t misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
{
    printk("misc_write\n ");
    return 0;
}
 
/**
 * @name: misc_release
 * @test: 当设备文件被关闭时内核会调用这个操作，当然这也可以不实现，函数默认为NULL。关闭设备永远成功。
 * @msg: 
 * @param {structinode} *inode 设备节点
 * @param {structfile} *file filefile结构体
 * @return {0}
 */
int misc_release(struct inode *inode,struct file *file){
    printk("hello misc_release bye bye \n");
    return 0;
}
 
/**
 * @name: misc_open
 * @test: 在操作设备前必须先调用open函数打开文件，可以干一些需要的初始化操作。
 * @msg: 
 * @param {structinode} *inode 设备节点
 * @param {structfile} *file filefile结构体
 * @return {0}
 */
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
 
//文件操作集
struct file_operations misc_fops =
    {
        .owner = THIS_MODULE,
        .open = misc_open,
        .release = misc_release,
        .read = misc_read,
        .write = misc_write,
};
//miscdevice结构体
struct miscdevice misc_dev =
    {
        .minor = MISC_DYNAMIC_MINOR,
        .name = "hello_misc",
        .fops = &misc_fops,
};
 
static int misc_init(void)
{
    int ret;
    ret = misc_register(&misc_dev); //注册杂项设备
    if (ret < 0)
    {
        printk("misc registe is error \n");
    }
    printk("misc registe is succeed \n");
    return 0;
}
static void misc_exit(void)
{
    misc_deregister(&misc_dev); //卸载杂项设备
    printk(" misc gooodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```
#### app.c编写
> app.c需使用**交叉编译器**编译
- 完整代码
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
int main(int argc,char *argv[])
{
    int fd; //定义一个句柄
    char buf[64] = {0};
    fd = open("/dev/hello_misc",O_RDWR);//打开设备节点
    if(fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    write(fd,buf,sizeof(buf));
    close(fd);
    return 0;
}
```
### 编译加载运行
#### 编译驱动
- 修改`Makefile`文件
```shell
obj-m += file_operation.o
KDIR:=/home/topeet/linux/linux-imx
PWD?=$(shell pwd)
all:
	make -C $(KDIR) M=$(PWD) modules ARCH=arm64
clean:
	make -C $(KDIR) M=$(PWD) clean
```

- 输入`./build.sh`编译

#### 加载驱动
```shell
insmod file_operation.ko    

ls /dev/hello_misc
```

#### 运行应用程序
```shell
./app
```

> 如果我们在驱动文件`file_operations`中没有设置对应的`read`，应用层调用`read`时什么也不会发生

### 应用层和内核层传递数据
#### 内核相关辅助函数
> 应用层和内核层不能直接传递数据

> 只能在内核中被调用

- 借助对应函数
```c 
/**
 * @brief 从用户空间复制数据到内核空间。
 *
 * 该函数安全地将数据从用户空间复制到内核空间。
 * 它用于确保数据正确复制，不会导致内存损坏或安全问题。
 *
 * @param to 内核空间目标缓冲区的指针。
 * @param from 用户空间源缓冲区的指针。
 * @param n 要复制的字节数。
 * @return 成功时返回复制的字节数，出错时返回负的错误代码。
 */
static inline long copy_from_user(void *to, const void __user *from, unsigned long n);

/**
 * @brief 从内核空间复制数据到用户空间。
 *
 * 该函数安全地将数据从内核空间复制到用户空间。
 * 它用于确保数据正确复制，不会导致内存损坏或安全问题。
 *
 * @param to 用户空间目标缓冲区的指针。
 * @param from 内核空间源缓冲区的指针。
 * @param n 要复制的字节数。
 * @return 成功时返回复制的字节数，出错时返回负的错误代码。
 */
static inline long copy_to_user(void __user *to, const void *from, unsigned long n);
```

#### 应用层从内核中读取
- 修改驱动程序实现应用层读取数据
```c
#include <linux/init.h>       //初始化头文件
#include <linux/module.h>     //最基本的文件，支持动态添加和卸载模块。
#include <linux/miscdevice.h> //注册杂项设备头文件
#include <linux/uaccess.h>
#include <linux/fs.h>
/**
 * @name: misc_read
 * @test: 从设备中读取数据，当用户层调用函数read时，对应的，内核驱动就会调用这个函数。
 * @msg: 
 * @param {structfile} *file file结构体
 * @param {char__user} *ubuf 这是对应用户层的read函数的第二个参数void *buf
 * @param {size_t} size 对应应用层的read函数的第三个参数
 * @param {loff_t} *loff_t 这是用于存放文件的偏移量的，回想一下系统编程时，读写文件的操作都会使偏移量往后移。
 * @return {*} 当返回正数时，内核会把值传给应用程序的返回值。一般的，调用成功会返回成功读取的字节数。
                如果返回负数，内核就会认为这是错误，应用程序返回-1
 */
ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    char kbuf[] = "hehe";
    if (copy_to_user(ubuf, kbuf, strlen(kbuf)) != 0)
    {
        printk("copy_to_user error\n ");
        return -1;
    }
    printk("misc_read\n ");
    return 0;
}
/**
 * @name: misc_write
 * @test: 往设备写入数据，当用户层调用函数write时，对应的，内核驱动就会调用这个函数。
 * @msg: 
 * @param {structfile} * filefile结构体
 * @param {constchar__user} *ubuf 这是对应用户层的write函数的第二个参数const void *buf
 * @param {size_t} size 对应用户层的write函数的第三个参数count。
 * @param {loff_t} *loff_t 这是用于存放文件的偏移量的，回想一下系统编程时，读写文件的操作都会使偏移量往后移。
 * @return {*} 当返回正数时，内核会把值传给应用程序的返回值。一般的，调用成功会返回成功读取的字节数。
                如果返回负数，内核就会认为这是错误，应用程序返回-1。
 */
ssize_t test_misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
{
    char kbuf[64] = {0};
    if (copy_from_user(kbuf, ubuf, size) != 0)
    {
        printk("copy_from_user error\n ");
        return -1;
    }
    printk("kbuf is %s\n ", kbuf);
    return 0;
}
/**
 * @name: misc_release
 * @test: 当设备文件被关闭时内核会调用这个操作，当然这也可以不实现，函数默认为NULL。关闭设备永远成功。
 * @msg: 
 * @param {structinode} *inode 设备节点
 * @param {structfile} *file filefile结构体
 * @return {0}
 */
int misc_release(struct inode *inode, struct file *file)
{
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
/**
 * @name: misc_open
 * @test: 在操作设备前必须先调用open函数打开文件，可以干一些需要的初始化操作。
 * @msg: 
 * @param {structinode} *inode 设备节点
 * @param {structfile} *file filefile结构体
 * @return {0}
 */
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
//文件操作集
struct file_operations misc_fops = {
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = test_misc_write
};
//miscdevice结构体
struct miscdevice misc_dev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "hello_misc",
    .fops = &misc_fops,
};
static int misc_init(void)
{
    int ret;
    ret = misc_register(&misc_dev); //注册杂项设备
    if (ret < 0)
    {
        printk("misc registe is error \n");
    }
    printk("misc registe is succeed \n");
    return 0;
}
static void misc_exit(void)
{
    misc_deregister(&misc_dev); //卸载杂项设备
    printk(" misc gooodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```

- 修改应用程序
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
int main(int argc,char *argv[])
{
    int fd;
    char buf[64] = "12345";
    fd = open("/dev/hello_misc",O_RDWR);//打开设备节点
    if(fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    //read(fd,buf,sizeof(buf));
    
    write(fd,buf,sizeof(buf)); //向内核层写数据
    //printf("buf is %s\n",buf);
    
    close(fd);
    return 0;
}
```

#### 应用层写入内核层
- 修改驱动文件
```c
#include <linux/init.h>       //初始化头文件
#include <linux/module.h>     //最基本的文件，支持动态添加和卸载模块。
#include <linux/miscdevice.h> //注册杂项设备头文件
#include <linux/uaccess.h>
#include <linux/fs.h>
/**
 * @name: misc_read
 * @test: 从设备中读取数据，当用户层调用函数read时，对应的，内核驱动就会调用这个函数。
 * @msg: 
 * @param {structfile} *file file结构体
 * @param {char__user} *ubuf 这是对应用户层的read函数的第二个参数void *buf
 * @param {size_t} size 对应应用层的read函数的第三个参数
 * @param {loff_t} *loff_t 这是用于存放文件的偏移量的，回想一下系统编程时，读写文件的操作都会使偏移量往后移。
 * @return {*} 当返回正数时，内核会把值传给应用程序的返回值。一般的，调用成功会返回成功读取的字节数。
                如果返回负数，内核就会认为这是错误，应用程序返回-1
 */
ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    char kbuf[] = "hehe";
    if (copy_to_user(ubuf, kbuf, strlen(kbuf)) != 0)
    {
        printk("copy_to_user error\n ");
        return -1;
    }
    printk("misc_read\n ");
    return 0;
}
/**
 * @name: misc_write
 * @test: 往设备写入数据，当用户层调用函数write时，对应的，内核驱动就会调用这个函数。
 * @msg: 
 * @param {structfile} * filefile结构体
 * @param {constchar__user} *ubuf 这是对应用户层的write函数的第二个参数const void *buf
 * @param {size_t} size 对应用户层的write函数的第三个参数count。
 * @param {loff_t} *loff_t 这是用于存放文件的偏移量的，回想一下系统编程时，读写文件的操作都会使偏移量往后移。
 * @return {*} 当返回正数时，内核会把值传给应用程序的返回值。一般的，调用成功会返回成功读取的字节数。
                如果返回负数，内核就会认为这是错误，应用程序返回-1。
 */
ssize_t test_misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
{
    char kbuf[64] = {0};
    if (copy_from_user(kbuf, ubuf, size) != 0)
    {
        printk("copy_from_user error\n ");
        return -1;
    }
    printk("kbuf is %s\n ", kbuf);
    return 0;
}
/**
 * @name: misc_release
 * @test: 当设备文件被关闭时内核会调用这个操作，当然这也可以不实现，函数默认为NULL。关闭设备永远成功。
 * @msg: 
 * @param {structinode} *inode 设备节点
 * @param {structfile} *file filefile结构体
 * @return {0}
 */
int misc_release(struct inode *inode, struct file *file)
{
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
/**
 * @name: misc_open
 * @test: 在操作设备前必须先调用open函数打开文件，可以干一些需要的初始化操作。
 * @msg: 
 * @param {structinode} *inode 设备节点
 * @param {structfile} *file filefile结构体
 * @return {0}
 */
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
//文件操作集
struct file_operations misc_fops = {
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = test_misc_write
};
//miscdevice结构体
struct miscdevice misc_dev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "hello_misc",
    .fops = &misc_fops,
};
static int misc_init(void)
{
    int ret;
    ret = misc_register(&misc_dev); //注册杂项设备
    if (ret < 0)
    {
        printk("misc registe is error \n");
    }
    printk("misc registe is succeed \n");
    return 0;
}
static void misc_exit(void)
{
    misc_deregister(&misc_dev); //卸载杂项设备
    printk(" misc gooodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```

- 修改应用程序
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
int main(int argc,char *argv[])
{
    int fd;
    char buf[64] = "12345";
    fd = open("/dev/hello_misc",O_RDWR);//打开设备节点
    if(fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    //read(fd,buf,sizeof(buf));
    
    write(fd,buf,sizeof(buf)); //向内核层写数据
    //printf("buf is %s\n",buf);
    
    close(fd);
    return 0;
}
```
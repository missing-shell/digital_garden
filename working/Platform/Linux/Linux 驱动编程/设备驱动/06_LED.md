### 简介
#### 目的
- 使用杂项设备完成一个LED驱动
- 完成一个上层测试应用
- 上层应用传入`1`打开LED，传入`0`关闭LED
#### 需求分析
- 按照[[03_MISC驱动#注册通用思路|杂项设备驱动的注册流程]]注册一个`LED`杂项设备结构体
- 使用设备节点关联硬件设备
- 完成相应`read`和`write`函数
- [[04_Linux用户层和内核层#应用层和内核层传递数据|应用层向内核传递参数]]
#### 硬件分析
> 在驱动开发调试过程，我们可以通过 `io` 命令访问寄存器，如需判断` iomux`， `gpio direction`/电平，提高/减小驱动强度，使能施密特触发，这是一个高效又准确的调试手段。

- 像复用关系的寄存器，电气属性的寄存器已经设置完成
- 直接设置*数据*寄存器和*方向*寄存器
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
    /*应用程序传入数据到内核空间，然后控制LED的逻辑，在此添加*/
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
    *vir_gpio1_io13_gdir |= (1 << 13);
    if (kbuf[0] == 1) //传入数据为1 ，LED亮
    {
        *vir_gpio1_io13 |= (1 << 13);
    }
    else if (kbuf[0] == 0)
    { //传入数据为0，LED灭
        *vir_gpio1_io13 &= ~(1 << 13);
    }
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
 * @Descripttion: 基于杂项设备的LED驱动
 */
#include <linux/init.h>            //初始化头文件
#include <linux/module.h>          //最基本的文件，支持动态添加和卸载模块。
#include <linux/miscdevice.h>      //包含了miscdevice结构的定义及相关的操作函数。
#include <linux/fs.h>              //文件系统头文件，定义文件表结构（file,buffer_head,m_inode等）
#include <linux/uaccess.h>         //包含了copy_to_user、copy_from_user等内核访问用户进程内存地址的函数定义。
#include <linux/io.h>              //包含了ioremap、iowrite等内核访问IO内存等函数的定义。
#include <linux/kernel.h>          //驱动要写入内核，与内核相关的头文件
#define GPIO1_IO13 0x30200000      //led数据寄存器物理地址
#define GPIO1_IO13_GDIR 0x30200004 //led数据寄存器方向寄存器物理地址
unsigned int *vir_gpio1_io13;      //存放映射完的虚拟地址的首地址
unsigned int *vir_gpio1_io13_gdir; //存放映射完的虚拟地址的首地址
 
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
    /*应用程序传入数据到内核空间，然后控制LED的逻辑，在此添加*/
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
    *vir_gpio1_io13_gdir |= (1 << 13);
    if (kbuf[0] == 1) //传入数据为1 ，LED亮
    {
        *vir_gpio1_io13 |= (1 << 13);
    }
    else if (kbuf[0] == 0)
    { //传入数据为0，LED灭
        *vir_gpio1_io13 &= ~(1 << 13);
    }
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
    .write = misc_write,
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
    //注册杂项设备
    ret = misc_register(&misc_dev);
    if (ret < 0)
    {
        printk("misc registe is error \n");
    }
    printk("misc registe is succeed \n");
    //将物理地址转化为虚拟地址
    vir_gpio1_io13 = ioremap(GPIO1_IO13, 4);
    if (vir_gpio1_io13 == NULL)
    {
        printk("vir_gpio1_io13 ioremap is error \n");
        return EBUSY;
    }
    printk("vir_gpio1_io13 ioremap is ok \n");
 
    vir_gpio1_io13_gdir = ioremap(GPIO1_IO13_GDIR, 4);
    if (vir_gpio1_io13 == NULL)
    {
        printk("GPIO1_IO13_GDIR ioremap is error \n");
        return EBUSY;
    }
    printk("GPIO1_IO13_GDIR ioremap is ok \n");
 
    return 0;
}
static void misc_exit(void)
{
    //卸载杂项设备
    misc_deregister(&misc_dev);
    iounmap(vir_gpio1_io13);
    iounmap(vir_gpio1_io13_gdir);
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
    int fd;
    char buf[64] = {0};//定义buf缓存
    //打开设备节点
    fd = open("/dev/hello_misc",O_RDWR);
    if(fd < 0)
    {
        //打开设备节点失败
        perror("open error \n"); 
        return fd;
    }
    // atoi()将字符串转为整型，这里将第一个参数转化为整型后，存放在buf[0]中
    buf[0] = atoi(argv[1]);
    //把缓冲区数据写入文件中
    write(fd,buf,sizeof(buf));  
    printf("buf is %d\n",buf[0]); 
    close(fd);
    return 0;
}
```
### 编译加载运行
#### 编译驱动
- 修改`Makefile`文件
```shell
obj-m += led.o
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
insmod led.ko    

ls /dev/hello_misc
```

#### 运行应用程序
```shell
./app 1 #灯亮
./app 0 #灯灭
```


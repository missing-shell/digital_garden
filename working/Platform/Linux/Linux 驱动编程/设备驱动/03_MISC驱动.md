### 简介
#### MISC定义
> Linux MISC 是最简单的**字符设备**驱动。

- 当设备没法分类时，就采用MISC设备驱动。
- 注册相对字符设备更简单，比较适合功能简单的设备
- 因为比较简单，通常嵌套于`platform`总线驱动中，配合总线驱动达到更复杂的效果
- 查看系统杂项设备
	```shell
	cat /proc/misc
	```
#### 设备节点（设备文件）
- `dev`目录下都是生成的设备节点
- `Linux`中设备节点是通过`mknod`命令来创建的。
- 一个设备节点其实就是一个文件，`Linux`中称为设备文件。
- 在`Linux`中，所有的设备访问都是通过文件的方式，一般的数据文件称为普通文件，设备节点称为设备文件。
- 设备节点就是连接上层应用和底层驱动的桥梁，如下图所示。
![[设备节点.png]]
#### MISC优势
- 杂项设备相对字符设备的一个优势时，自动生成**设备节点**，
- `major=10`，主设备相同可以节省内核资源，`minor`通常从`0`开始
#### 主次设备号
- 设备号包含*主*设备号和*次*设备号
- 设备号是计算机识别设备的一种方式，驱动程序初始化，会注册它的驱动及对应的主设备号到系统中，相同的主设备号被视为同一类设备
- 主设备号在Linux系统里面是唯一的，次设备号不一定唯一。
- 驱动程序遍历设备时，每发现一个它能驱动的设备，就创建一个设备对象，并为其分配一个次设备号以区分同类设备下不同的设备。
- 查看主设备号
	```shell
		cat/proc/devices
	```
- `misc`设备使用`miscdevice`结构体表示，定义在`include/linux/miscdevice.h`中
```c
struct miscdevice  {
	int minor; //次设备号
	const char *name; //设备节点的名字
	const struct file_operations *fops; //文件操作集
	struct list_head list;
	struct device *parent;
	struct device *this_device;
	const struct attribute_group **groups;
	const char *nodename;
	umode_t mode;
};
```
- 必须指定`minor`,`name`和`fops`三个成员变量
- `minor`可以自定义，也可从系统预定义的子设备号中选择`linux/miscdevice.h`
- `name`是对应驱动的名称，注册成功后会在`/dev`目录下生成一个名为`name`的设备文件
- `fops`是这个设备的操作集合
- 注册`misc`设备
```c
int misc_register(struct miscdevice* misc)
```
- 注销`misc`设备
```c
int misc_deregister(struct miscdevice *misc)
```
#### file_opreation
- `file_operation`文件操作集定义在`include/linux/fs.h`目录下，由`device/char/misc.c`驱动核心层的`misc_fops`间接调用

> 设备驱动
- 设备驱动程序（`device driver`），简称驱动程序（`driver`），是一个允许高级(`High level`)计算机软件（`computer software`）与硬件（`hardware`）交互的程序
- 这种程序建立了一个硬件与硬件，或硬件与软件沟通的界面，经由主板上的总线（`bus`）或其它沟通子系统（`subsystem`）与硬件形成连接的机制，这样的机制使得硬件设备（`device`）上的数据交换成为可能。
- 点`led`灯的驱动，就是简单的io操作。
- `file_operations`结构体是访问驱动的函数，它的里面的每个结构体成员都对应一个调用
- 结构体中的成员函数是字符设备驱动程序设计的主体内容，这些函数实际会在应用程序进行`Linux`的`open()`、`write()`、`read()`、`close()`等系统调用时最终被内核调用。
- `file_operations`文件操作集在定义在`/home/topeet/linux/linux-imx/include/linux/fs.h`
- 结构体定义
```c
struct file_operations {
	struct module *owner;
	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
	ssize_t (*aio_read) (struct kiocb *, const struct iovec *, unsigned long, loff_t);
	ssize_t (*aio_write) (struct kiocb *, const struct iovec *, unsigned long, loff_t);
	int (*iterate) (struct file *, struct dir_context *);
	unsigned int (*poll) (struct file *, struct poll_table_struct *);
	long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
	long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
	int (*mmap) (struct file *, struct vm_area_struct *);
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *, fl_owner_t id);
	int (*release) (struct inode *, struct file *);
	int (*fsync) (struct file *, loff_t, loff_t, int datasync);
	int (*aio_fsync) (struct kiocb *, int datasync);
	int (*fasync) (int, struct file *, int);
	int (*lock) (struct file *, int, struct file_lock *);
	ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
	unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long,unsigned long, unsigned long);
	int (*check_flags)(int);
	int (*flock) (struct file *, int, struct file_lock *);
	ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
	ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
	int (*setlease)(struct file *, long, struct file_lock **);
	long (*fallocate)(struct file *file, int mode, loff_t offset,loff_t len);
	int (*show_fdinfo)(struct seq_file *m, struct file *f);
};
```
#### 注册通用思路
1. 填充`miscdevice`结构体
2. 填充`file_operations`结构体
3. 注册杂项设备并生成设备节点

### 示例编写
#### 编写驱动
- 填充`miscdevice`结构体
	```c
struct miscdevice misc_dev = {
	.minor = MISC_DYNAMIC_MINOR,
	.name = "hello_misc",
	.fops = &misc_fops,
};
```

- 填充`file_operation`结构体
```c
struct file_operations misc_fops={
	.owner = THIS_MODULE
};
```

> `#define THIS_MODULE (&__this_module)`
> 使用`THIS_MODULE`宏来引用`struct module`结构

- 注册杂项设备并生成设备节点
```c
static int misc_init(void){
	int ret;
	ret = misc_register(&misc_dev);             //注册杂项设备
	if(ret<0)                                   //判断杂项设备是否注册成功
	{
		printk("misc registe is error \n");     //打印杂项设备注册失败
	}
	printk("misc registe is succeed \n");       //打印杂项设备注册成功
	return 0;
}

/*在misc_exit（）函数中填充misc_deregister()函数注销杂项设备。*/
 static void misc_exit(void){
	misc_deregister(&misc_dev);                 //注销杂项设备
	printk("misc gooodbye! \n");                //打印杂项设备注销成功
}
```

- 完整代码
```c
/*
 * @Descripttion: 最简单的杂项设备驱动
 * @version: 1.0
 * @Author: topeet
 */
#include <linux/init.h>              //初始化头文件
#include <linux/module.h>            //最基本的文件，支持动态添加和卸载模块。
#include <linux/miscdevice.h>        /*注册杂项设备头文件*/
#include <linux/fs.h>                /*注册设备节点的文件结构体*/
struct file_operations misc_fops = { //文件操作集
	.owner = THIS_MODULE};
struct miscdevice misc_dev = {
	//杂项设备结构体
	.minor = MISC_DYNAMIC_MINOR, //动态申请的次设备号
	.name = "hello_misc",        //杂项设备名字是hello_misc
	.fops = &misc_fops,          //文件操作集
 
};
static int misc_init(void)
{ //在初始化函数中注册杂项设备
	int ret;
	ret = misc_register(&misc_dev);
	if (ret < 0)
	{
		printk("misc registe is error \n");
	}
	printk("misc registe is succeed \n");
	return 0;
}
static void misc_exit(void)
{ //在卸载函数中注销杂项设备
	misc_deregister(&misc_dev);
	printk(" misc gooodbye! \n");
}
module_init(misc_init);
module_exit(misc_exit);
MODULE_LICENSE("GPL");
```
#### 编译驱动
- 参照[[01_hello_world#驱动编译|helloworld驱动编译]]
#### 运行测试
- 加载驱动模块
- 每次*加载*驱动前需确保之前的驱动已经被*卸载*
``` shell
insmod misc.ko
```

- 查看注册的设备节点是否存在
```shell
ls /dev/h*
```

- 卸载驱动
```shell
rmmod misc
```
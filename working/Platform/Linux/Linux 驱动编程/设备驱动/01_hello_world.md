### 驱动组成
> 头文件
```c
#include <linux/init.h>   //包含宏定义的头文件
#include <linux/module.h> //包含初始化加载模块的头文件
```
> 驱动模块的入口和出口
```c
module_init();
module_exit();
```
> 声明信息-许可证
```c
MODULE_LICENSE("GPL");
```
> 功能实现
- 打印不能使用`printf`，因为内核无法使用`C`库函数。
- 使用`printk`打印

最简单的`Linux`内核模块
```c
#include <linux/init.h>   
#include <linux/module.h>  

static int hello_world_init(void)
{
    printk("Hello World\n");
    return 0;
}
static void hello_world_exit(void)
{
    printk("Goodbye World\n");
}
module_init(hello_world_init);
module_exit(hello_world_exit);

MODULE_LICENSE("GPL");
```
### 驱动编译
#### 步骤一：创建`Makefile`文件
```make
obj-m += helloworld.o  //-m 把驱动编译为模块
KDIR:=/home/topeet/linux/linux-imx//内核源码的实际路径
PWD?=$(shell pwd)//获取当前目录的变量
all:
	make -C $(KDIR) M=$(PWD) modules//进入内核源码的路径，然后编译为模块
clean:
	make -C $(KDIR) M=$(PWD) clean//清除编译文件
```
#### 步骤二：设置交叉编译器
```shell
#!/bin/bash
#使用bash来执行此脚本
#clear screen
clear
echo ""
echo -e "\033[32m **** 让我们开始编译驱动程序！ **** \033[0m"
echo ""
#set env
. /opt/fsl-imx-xwayland/4.14-sumo/environment-setup-aarch64-poky-linux
export ARCH=arm64
unset LDFLAGS
 
#clean cache files
make clean
 
#make 
make
 
#result
#\033[32m set color 
#\033[0m  clear color
result=$(ls |grep *.ko)
if [ "$result" = "" ];then 
    echo ""
    echo -e "\033[5m\033[31m **** 驱动程序编译失败！报错原因请参考上述提示 ****\033[0m"
    echo ""
else 
    echo ""
    echo -e "\033[32m **** 驱动程序编译成功！ **** \033[0m"
    echo ""
fi
```
#### 步骤三 驱动加载
1. 设置权限
```bash
 chmod 777 -R *
```
2. 运行脚本`./build.sh`
3. 编译生成`helloworld.ko`目标文件
4. 使用`nfs`传输到开发板上
5. 使用`insmod helloworld.ko`*加载驱动*
6. 使用`rmmod helloworld`*卸载驱动*

## 概念
> 是一种**异步**的事件处理机制，可以提高系统的并发处理能力。

当硬件设备发生某种事件，如`I/O`完成、定时器中断等，会*触发中断*，并切换到中断上下文执行中断处理程序。

*proc 文件系统*：一种内核空间和用户空间进行通信的机制，可以用来查看内核的数据结构，或者用来动态修改内核的配置。
- `/proc/softirqs`：提供软中断的运行情况；
- `/proc/interrupts`：提供硬中断的运行情况。
### 硬中断与软中断
*硬中断*：由硬件产生。
- 每个设备或设备集都有它自己的`IRQ`（中断请求）。基于`IRQ`，`CPU`可以将相应的请求分发到对应的硬件驱动上。
- 可以**直接中断**CPU，引起内核中相关的代码被触发。

*软中断*：软中断仅与**内核**相关，由当前正在运行的进程所产生，以**内核线程**的方式运行。
- 硬件中断处理程序的下半部。
- 包括*网络收发、定时、调度、RCU 锁*等各种类型，这些请求会调用内核中可以调度`I/O`发生的程序。
- 不会直接中断`CPU`，也只有当前*正在运行*的代码（或进程）才会产生软中断。
### 查看软中断和内核线程
1. 查看各种类型软中断在不同 CPU上的累积运行次数：
```bash
cat /proc/softirqs
```
软中断的类型：对应第1列，包含了10个类别，分别对应不同的工作类型。比如说`NET_RX `表示网络接收中断，而 `NET_TX `表示网络发送中断。

同一种软中断类型在不同`CPU`上的分布情况：对应每一行，正常情况下，同一种中断类型在不同CPU上的累计次数基本在同一个数量级。但是也有例外，比如`TASKLET`

2. 如何查看软中断内核线程的运行状况？
```bash
ps -ef|grep softirq
```
线程的名字外面都有中括号，这说明 ps 无法获取它们的命令行参数 （`cmline`）。

一般来说，`ps` 的输出中，名字括在中括号里的，一般都是内核线程。

3. 如何查看硬中断运行情况
```bash
cat /proc/interrupts 
```
#### 相关工具
1. `sar `是一个系统活动报告工具，既可以实时查看系统的当前活动，又可以配置保存和报告历史统计数据。
```bash
sar -n DEV 1 #表示显示网络收发的报告，间隔1秒输出一组数据
```
第1列：表示报告的时间
第2列：`IFACE` 表示网卡
第3，4列：`rxpck/s` 和 `txpck/s` 分别表示每秒接收、发送的网络帧数，也就是 `PPS`
第5，6列：`rxkB/s` 和 `txkB/s` 分别表示每秒接收、发送的千字节数，也就是 `BPS`。

2. `tcpdump` 是一个常用的网络抓包工具，常用来分析各种网络问题。
```bash
tcpdump -i eth0 -n tcp port 80  
```
- `-i eth0` 只抓取`eth0`网卡
- `-n`不解析协议名和主机名
- `tcp port 80`表示只抓取`tcp`协议并且端口号为`80`的网络帧
#### tasklet定义
- `TASKLET`是最常用的软中断实现机制，每个`TASKLET`只会*运行一次*就会结束，并且只在调用它的函数所在的`CPU`上运行，不能并行而只能**串行执行**。
- 多个不同类型的`TASKLET`可以并行在多个`CPU`上。
- **软中断**是*静态*，只能支持有限的几种软中断类型，一旦内核编译好之后就不能改变；而`TASKLET`灵活很多，可以通过*添加内核模块*的方式在运行时修改。
### 中断处理大致流程
1. *硬件触发中断*：中断可以通过硬件设备（如时钟、外部设备、异常等）发出中断请求信号。
2. *中断请求处理*：当硬件设备发出中断请求信号后，CPU会检查中断请求线。如果存在中断请求，CPU会立即响应并暂停当前正在执行的程序。
3. *中断处理程序调用*：当中断发生时，CPU会根据中断号或中断向量表中的映射关系，找到对应的中断处理程序的入口地址。保存当前中断上下文，然后跳转到中断处理程序的入口地址。
4. *中断处理*：中断处理程序根据中断类型或中断号，执行相应的处理逻辑。
5. *恢复执行*：中断处理程序执行完毕后，恢复中断上下文，然后继续执行被中断的程序。
### 中断上下部
> 中断服务程序分为两部分：上半部-下半部 。

硬中断（上半部）是会*打断* CPU 正在执行的任务，然后立即执行中断处理程序，而软中断（下半部）是以内核线程的方式执行，并且每一个 CPU 都对应一个软中断内核线程。

上半部用来快速处理中断，一般会暂时关闭中断请求，主要负责处理跟硬件紧密相关或者时间敏感的事情。避免因中断执行时间过长导致**中断丢失**。
#### 上半部 硬中断
> 响应速度快

- **响应中断**，读取寄存器中断状态，清除中断标志
- 把设备驱动程序中中断处理例程的下半部挂到设备的下半部执行队列中去。
- *继续等待新的中断到来*。
#### 下半部 软中断
- 处理中断的剩余大部分任务，可以被新的中断*打断*。
- 通常以**内核线程**的方式运行。

| 机制        | 上下文 | 复杂度 | 执行性能 | 顺序执行保障          |
| --------- | --- | --- | ---- | --------------- |
| 软中断       | 中断  | 高   | 好    | 没有              |
| `tasklet` | 中断  | 中   | 中    | 同类型不能同时执行       |
| 工作队列      | 进程  | 低   | 差    | 没有（和进程上下文一样被调度） |
#### 确定代码执行位置
- 如果要处理的内容不希望被其他中断*打断*，那么可以放到上半部。
- 如果要处理的任务对*时间敏感*，可以放到上半部。
- 如果要处理的任务与*硬件相关*，可以放到上半部
- 除了上述三点以外的其他任务，优先考虑放到下半部。
### 中断上下文
> **中断上下文**是指当发生==中断==或==异常==事件时，硬件或操作系统内核自动保存当前被中断程序的执行现场，并切换到中断处理程序执行的上下文环境。

- 中断上下文包含了被中断程序的*寄存器状态*、*堆栈指针*、*中断原因*等信息。
- 在中断处理程序执行过程中，*保存和恢复中断*程序的上下文是必要的，以确保被中断程序的执行能够正确继续。
### 中断与异常的区别
### 中断中的资源共享方式
> 中断处理程序不允许可能导致睡眠的操作。

- *自旋锁 (Spinlocks)*：自旋锁是一种常用的内核同步原语，尤其适用于保护中断上下文中的共享资源。
- *原子操作 (Atomic Operations)*：对于一些简单的共享资源，可以使用原子操作（如 `atomic_add`, `atomic_sub`, `atomic_cmpxchg` 等）来避免显式的加锁。
- *工作队列*：通过软中断和任务队列可以将某些非紧急的中断处理推迟到**进程上下文**中，从而避免在处理中断时做过多的资源共享操作。
### 相关函数
#### irq_of_parse_and_map()
```c
/*
 * irq_of_parse_and_map - Parse device tree interrupt properties
 * @dev: Pointer to the device node from which to parse the interrupts
 * @index: Index into the interrupts property
 *
 * Description: This function is used to parse the interrupt properties from
 * the device tree and map them to the corresponding device interrupts.
 * Returns: The irq number if successful, otherwise a negative error code.
 */

unsigned int irq_of_parse_and_map(struct device_node *dev, int index);
```
#### gpio_to_irq()
```c
/*
 * gpio_to_irq - Get the IRQ number for a given GPIO
 * @gpio: The GPIO number for which to get the IRQ number
 *
 * Description: This function is used to obtain the IRQ number associated with
 * a given GPIO pin.
 * Returns: The irq number if successful, otherwise a negative error code.
 */

int gpio_to_irq(unsigned int gpio);
```
#### request_irq()
```c
/*
 * request_irq - Allocate an interrupt line
 * @irq: Interrupt number
 * @handler: Function to be called when the IRQ occurs
 * @flags: Interrupt type flags
 * @name: An ascii string for the claimant
 * @dev: A class-specific pointer to be passed to the handler
 *
 * Description: This function is used to request an interrupt line, specifying
 * the interrupt number, a handler function, type flags, a name for the claimant,
 * and a class-specific pointer to be passed to the handler.
 * Returns: Zero if successful, else an error code.
 */

int request_irq(unsigned int irq, irq_handler_t handler, unsigned long flags,
                const char *name, void *dev);
```
#### free_irq()
```c
/*
 * free_irq - Free an interrupt line
 * @irq: Interrupt number
 * @dev: Class-specific pointer passed to the handler
 *
 * Description: This function is used to free an interrupt line that was
 * previously allocated with request_irq().
 */

void free_irq(unsigned int irq, void *dev);
```
#### irqreturn_t
```c
/*
 * irq_handler_t - Interrupt handler type
 * @irq: The irq number that is interrupting
 * @dev_id: The class-specific pointer passed to the handler
 *
 * Description: This is the prototype for the interrupt handler function.
 * It will be called when an interrupt occurs.
 * Returns: An irqreturn_t value indicating whether the interrupt was
 * handled or not.
 */

irqreturn_t (*irq_handler_t)(int irq, void *dev_id);
```
#### enable_irq()
```c
/*
 * enable_irq - Enable an interrupt
 * @irq: Interrupt number
 *
 * Description: This function is used to enable an interrupt, allowing it to
 * generate interrupts again.
 */

void enable_irq(unsigned int irq);
```
#### disable_irq()
```c
/*
 * disable_irq - Disable an interrupt
 * @irq: Interrupt number
 *
 * Description: This function is used to disable an interrupt, preventing it
 * from generating further interrupts. It will not return until the current
 * handler has finished execution.
 */

void disable_irq(unsigned int irq);
```
#### disable_irq_nosync()
```c
/*
 * disable_irq_nosync - Disable an interrupt without waiting for handlers
 * @irq: Interrupt number
 *
 * Description: This function is used to disable an interrupt, preventing it
 * from generating further interrupts. It will return immediately without
 * waiting for any current handler to finish execution.
 */

void disable_irq_nosync(unsigned int irq);
```
### 设备树中的中断节点
> 配置中断需先在设备树中配置好**中断属性**信息
- 设备树是用来描述硬件信息的，`Linux`内核通过设备树配置的*中断属性*来配置中断功能。
- 对于中断控制器而言，在`linux/linux-imx/arch/arm64/boot/dts/freescale/xxx-evk.dtsii`文件，其中的 `gic`节点就是`IMX8MM`的==中断控制器节点==
``` c
gic: interrupt-controller@38800000 {
		compatible = "arm,gic-v3";
		reg = <0x0 0x38800000 0 0x10000>, /* GIC Dist */
		      <0x0 0x38880000 0 0xC0000>; /* GICR (RD_base + SGI_base) */
		#interrupt-cells = <3>;
		interrupt-controller;
		interrupts = <GIC_PPI 9 IRQ_TYPE_LEVEL_HIGH>;
		interrupt-parent = <&gic>;
	};
```
- `IMX8MM`使用中断控制器是 `gic-v3`。
- `gic-v3` 是 `ARM Generic Interrupt Controller, version 3` 的缩写，是一款 `ARM` 出品的*通用中断控制器*。
- `AArch64 SMP` 内核通常与 `GICv3` 搭配使用，`GICv3` 提供了专用外设中断（`PPI`），共享外设中断（`SPI`），软件生成的中断（`SGI`）和特定于区域的外设中断（`LPI`）。
- `#interrupt-cells = <3>` 表明 `Interrupt client devices` 需要用 `3`个*单元*才能确定引用的中断，例如 `interrupts = <GIC_PPI 9 IRQ_TYPE_LEVEL_HIGH>`；`interrupt-controller `节点为*空*，表示当前节点是中断控制器

> 对于 `gpio` 来说，`gpio` 节点也可以作为**中断控制器**，比如 `imx6ull.dtsi` 文件中的 `gpio5` 节点
```c
gpio5 : gpio @020ac000{
compatible = "fsl,imx6ul-gpio", "fsl,imx35-gpio";
reg = <0x020ac000 0x4000>;
	interrupts = <GIC_SPI 74 IRQ_TYPE_LEVEL_HIGH>, <GIC_SPI 75 IRQ_TYPE_LEVEL_HIGH>;
	 gpio-controller;
	 #gpio-cells = <2>;
	 interrupt-controller;
	 #interrupt-cells = <2>;
	};
```
- 第四行中`interrupts` 描述中断源信息，对于 `gpio5` 来说一共有两条信息，中断类型都是 `SPI`，触发电平都是 `IRQ_TYPE_LEVEL_HIGH`。不同之处在于*中断源*，一个是 `74`，一个是 `75`
- 从手册中可知`GPIO5` 一共用了 `2` 个中断号，一个是 `74`，一个是 `75`。其中 `74` 对应 `GPIO5_IO00~GPIO5_IO15`低`16`个 `IO`，`75`对应`GPIO5_IO16~GPIOI5_IO31`这高`16`位`IO`。
#### 与中断有关的设备树属性
- `#interrupt-cells`，指定中断源的信息`cells`个数。 
- `interrupt-controller`，表示当前节点为中断控制器 
- `interrupts`，指定中断号，*触发方式*等。 
- `interrupt-parent`，指定父中断，也就是*中断控制器*
#### 示例
```c
ft5x06_ts@38 {
        compatible = "edt,edt-ft5x06";
        reg = <0x38>;
        pinctrl-names = "defaults";
        pinctrl-0 = <&pinctrl_ft5x06_int>;
        interrupt-parent = <&gpio1>;
        interrupts = <15 2>;
        status = "okay";
}；
pinctrl_ft5x06_int: ft5x06_int {
                        fsl,pins = <
                                /*MX8MM_IOMUXC_GPIO1_IO09_GPIO1_IO9               0x159*/
                                MX8MM_IOMUXC_GPIO1_IO15_GPIO1_IO15                0x159
                                MX8MM_IOMUXC_SAI5_RXD2_GPIO3_IO23                 0x41
                        >;
                };
```
- 首先使用`pinctrl`和`gpio`子系统把这个引脚设置为`gpio`功能，并将引脚设置成*输入*。
- 使用`interrupt-parent`和`interrupts`属性来描述中断。
- `interrupt-parent`的属性值是`gpio1`，也就是使用`gpio1`中断控制器，为什么是`gpio1`呢，因为引脚使用的是`gpio1`里面的`io15`。
- 第一个`cells`的`15`表示`GPIO1`组的`15`号`IO`。`2`表示*下降沿*有效。
- `interrupts`属性设置的是*中断源*，为什么里面是两个`cells`呢，因为在`gpio1`这个中断控制器里面`#interrupt-cells`的值为`2`，见下图
```c
gpioc0@30200000 {
	compatible = "fsl,imx8mm-gpio", "fsl,imx35-gpio";
	reg = <0x0 0x30200000 0x0 0x10000>,
	interrupts=<GIC_SPI 64 IRQ_TYPE_LEVEL_HIGH>,
			   <GIC_SPI 65 IRQ_TYPE_LEVEL_HIGH>;  
	gpio-controller;
	#gpio-cells = <2>;
	interrupt-controller;
	#interrupt-cells = <2>;
};
```
> 在设备树中配置中断
- *第一步*：把管脚设置为`gpio`功能。
- *第二步*：使用`interrupt-parent`和`interrupts`**属性**来描述中断。
## 示例
### 编写驱动
> Makefile
```c
obj-m += driver.o
KDIR:=/home/topeet/linux/linux-imx
PWD?=$(shell pwd)
all:
	make -C $(KDIR) M=$(PWD) modules ARCH=arm64
clean:
	make -C $(KDIR) M=$(PWD) clean
```

- driver.c
```c
/*
 * @Author:topeet
 * @Description: 使用irq_of_parse_and_map函数来获取中断号
 */
#include <linux/init.h>
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/of_address.h>
#include <linux/of_irq.h>
 
#include <linux/gpio.h>
#include <linux/of_gpio.h>
#include <linux/interrupt.h>
//定义结构体表示我们的节点
struct device_node *test_device_node;
struct property *test_node_property;
//要申请的中断号
int irq;
// GPIO 编号
int gpio_nu;
 
/**
 * @description: 中断处理函数test_key
 * @param {int} irq ：要申请的中断号
 * @param {void} *args ：
 * @return {*}IRQ_HANDLED
 */
irqreturn_t test_key(int irq, void *args)
{
    printk("test_key \n");
    return IRQ_RETVAL(IRQ_HANDLED);
}
/****************************************************************************************
 * @brief led_probe : 与设备信息层（设备树）匹配成功后自动执行此函数，
 * @param inode : 文件索引
 * @param file  : 文件
 * @return 成功返回 0           
 ****************************************************************************************/
int led_probe(struct platform_device *pdev)
{
    int ret = 0;
    // 打印匹配成功进入probe函数
    printk("led_probe\n");
    test_device_node = of_find_node_by_path("/test");
    if (test_device_node == NULL)
    {
        //查找节点失败则打印信息
        printk("of_find_node_by_path is error \n");
        return -1;
    }
    gpio_nu = of_get_named_gpio(test_device_node, "gpios", 0);
 
    if (gpio_nu < 0)
    {
        printk("of_get_namd_gpio is error \n");
        return -1;
    }
    //设置GPIO为输入模式
    gpio_direction_input(gpio_nu);
    //获取GPIO对应的中断号
    //irq = gpio_to_irq(gpio_nu);
    irq = irq_of_parse_and_map(test_device_node, 0);
    printk("irq is %d \n", irq);
    /*申请中断，irq:中断号名字  
     test_key：中断处理函数
     IRQF_TRIGGER_RISING：中断标志，意为上升沿触发
     "test_key"：中断的名字
     */
    ret = request_irq(irq, test_key, IRQF_TRIGGER_RISING, "test_key", NULL);
    if (ret < 0)
    {
        printk("request_irq is error \n");
        return -1;
    }
    return 0;
}
 
int led_remove(struct platform_device *pdev)
{
    printk("led_remove\n");
    return 0;
}
const struct platform_device_id led_idtable = {
    .name = "keys",
};
const struct of_device_id of_match_table_test[] = {
    {.compatible = "keys"},
    {},
};
struct platform_driver led_driver = {
    //3. 在led_driver结构体中完成了led_probe和led_remove
    .probe = led_probe,
    .remove = led_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "led_test",
        .of_match_table = of_match_table_test},
    //4 .id_table的优先级要比driver.name的优先级要高，优先与.id_table进行匹配
    .id_table = &led_idtable};
 
/**
 * @description: 模块初始化函数
 * @param {*}
 * @return {*}
 */
static int led_driver_init(void)
{
    //1.我们看驱动文件要从init函数开始看
    int ret = 0;
    //2.在init函数里面注册了platform_driver
    ret = platform_driver_register(&led_driver);
    if (ret < 0)
    {
        printk("platform_driver_register error \n");
    }
    printk("platform_driver_register ok \n");
    return 0;
}
 
/**
 * @description: 模块卸载函数
 * @param {*}
 * @return {*}
 */
static void led_driver_exit(void)
{
    free_irq(irq, NULL);
    platform_driver_unregister(&led_driver);
    printk("goodbye! \n");
}
module_init(led_driver_init);
module_exit(led_driver_exit);
 
MODULE_LICENSE("GPL");
```
### 编译运行
- 加载驱动模块`insmod driver.ko`
- 验证是否成功申请中断
```shell
cat /proc/interrupts | grep test_key
```
- 查看中断次数
```shell
cat /proc/irq/196/spurious
```
### 方案优化
上面的示例是使用函数`gpio_to_irq`来获取中断号的，接下来通过在*设备树*文件里面使用属性`interrupt-parent`和`interrupts`来获取中断号
```c
#include "fsl-imx8mm.dtsi"
/{
	model="TOPEET i.MX8MM EVK board";
	compatible="fsl,imx8mm-evk","fsl,imx8mm";
	
	test:test {
	    compatible = "keys";
	    pinctrl-names = "default";
	    pinctrl-0 = <&pinctrl_gpio_keys>;
	    gpios = <&gpio4 31 GPIO_ACTIVE_LOW>;
	    interrupt-parent = <&gpio4>;
	    interrupt = <31 IRQ_TYPE_EDGE_RISING>;
	};
 };
```
- 重新编译源码，然后烧写设备树镜像
- 在开发板上使用`ls /proc/device-tree/test/`
- 修改`driver.c`
```c
/*
 * @Author:topeet
 * @Description: 使用gpio_to_irq函数来获取中断号
 */
#include <linux/init.h>
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/of_address.h>
#include <linux/of_irq.h>
 
#include <linux/gpio.h>
#include <linux/of_gpio.h>
#include <linux/interrupt.h>
//定义结构体表示我们的节点
struct device_node *test_device_node;
struct property *test_node_property;
//要申请的中断号
int irq;
// GPIO 编号
int gpio_nu;
 
/**
 * @description: 中断处理函数test_key
 * @param {int} irq ：要申请的中断号
 * @param {void} *args ：
 * @return {*}IRQ_HANDLED
 */
irqreturn_t test_key(int irq, void *args)
{
    printk("test_key \n");
    return IRQ_HANDLED;
}
/****************************************************************************************
 * @brief led_probe : 与设备信息层（设备树）匹配成功后自动执行此函数，
 * @param inode : 文件索引
 * @param file  : 文件
 * @return 成功返回 0           
 ****************************************************************************************/
int led_probe(struct platform_device *pdev)
{
    int ret = 0;
    // 打印匹配成功进入probe函数
    printk("led_probe\n");
    test_device_node = of_find_node_by_path("/test");
    if (test_device_node == NULL)
    {
        //查找节点失败则打印信息
        printk("of_find_node_by_path is error \n");
        return -1;
    }
      gpio_nu = of_get_named_gpio(test_device_node, "gpios", 0);
    if (gpio_nu < 0)
    {
        printk("of_get_namd_gpio is error \n");
        return -1;
    }
    //设置GPIO为输入模式
    gpio_direction_input(gpio_nu);
    //获取GPIO对应的中断号
   // irq = gpio_to_irq(gpio_nu);
   irq =irq_of_parse_and_map(test_device_node,0);
    printk("irq is %d \n", irq);
    /*申请中断，irq:中断号名字  
     test_key：中断处理函数
     IRQF_TRIGGER_RISING：中断标志，意为上升沿触发
     "test_key"：中断的名字
     */
    ret = request_irq(irq, test_key, IRQF_TRIGGER_RISING, "test_key", NULL);
    if (ret < 0)
    {
        printk("request_irq is error \n");
        return -1;
    }
    return 0;
}
 
int led_remove(struct platform_device *pdev)
{
    printk("led_remove\n");
    return 0;
}
const struct platform_device_id led_idtable = {
    .name = "keys",
};
const struct of_device_id of_match_table_test[] = {
    {.compatible = "keys"},
    {},
};
struct platform_driver led_driver = {
    //3. 在led_driver结构体中完成了led_probe和led_remove
    .probe = led_probe,
    .remove = led_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "led_test",
        .of_match_table = of_match_table_test},
    //4 .id_table的优先级要比driver.name的优先级要高，优先与.id_table进行匹配
    .id_table = &led_idtable};
 
/**
 * @description: 模块初始化函数
 * @param {*}
 * @return {*}
 */
static int led_driver_init(void)
{
    //1.我们看驱动文件要从init函数开始看
    int ret = 0;
    //2.在init函数里面注册了platform_driver
    ret = platform_driver_register(&led_driver);
    if (ret < 0)
    {
        printk("platform_driver_register error \n");
    }
    printk("platform_driver_register ok \n");
    return 0;
}
 
/**
 * @description: 模块卸载函数
 * @param {*}
 * @return {*}
 */
static void led_driver_exit(void)
{
    free_irq(irq, NULL);
    platform_driver_unregister(&led_driver);
    printk("gooodbye! \n");
}
module_init(led_driver_init);
module_exit(led_driver_exit);
 
MODULE_LICENSE("GPL");
```
- 编译成功加载驱动，可以成功获得`irq`号
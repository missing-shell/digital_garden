## 定义
### 设备驱动的*分层*思想
> 在设备驱动方面，往往为同类的设备设计了一个框架，而框架中的核心层则实现了该设备通用的一些功能。
- 同样的，如果具体的设备不想使用核心层的函数，它可以重载之。
- 下述`core_funca`函数的实现中，会检查底层设备是否重载了`funca()`，如果重载了，则调用底层代码，否则直接调用通用层代码。
```c
return_type core_funca(xxx_device * bottom_dev, param1_type param1, param1_type param2)
{
	if (bottom_dev->funca)
	return bottom_dev->funca(param1, param2);
	/* 核心层通用的 funca 代码 */
	...
}
```

- 对于同类设备而言，操作流程一致，都要经过“通用代码 A、底层 ops1、通用代码 B、底层 ops2、通用代码 C、底层 ops3”这几步，分层设计明显带来的好处是，对于通用代码 A、B、C，具体的底层驱动不需要再实现，而仅仅只关心其底层的操作 ops1、ops2、ops3。这样的分层化设计在 Linux 的 `input`、`RTC`、`MTD`、`I2C`、`SPI`、`TTY`、`USB` 等诸多设备驱动类型中屡见不鲜。
```c
copyreturn_type core_funca(xxx_device * bottom_dev, param1_type param1, param1_type param2)
{
	/*通用的步骤代码 A */
	...
	bottom_dev->funca_ops1();
	/*通用的步骤代码 B */
	...
	bottom_dev->funca_ops2();
	/*通用的步骤代码 C */
	...
	bottom_dev->funca_ops3();
}
```
### 主机驱动与外设驱动*分离*思想
- 假设我们要通过 SPI 总线访问某外设，在这个访问过程中，要通过操作 CPU XXX 上的 SPI 控制器的寄存器来达到访问 SPI 外设 YYY 的目的，最简单的方法是：
```c
copyreturn_type xxx_write_spi_yyy(...)
{
	xxx_write_spi_host_ctrl_reg(ctrl);
	xxx_ write_spi_host_data_reg(buf);
	while(!(xxx_spi_host_status_reg()&SPI_DATA_TRANSFER_DONE));
	...
}
```
- 当外设`yyy`用在不同`CPU`上时需要不同的驱动，这是不可接收的

> 分离主机与外设的驱动
## `Platform`平台驱动模型
### 前提
> 在 Linux 2.6 以后的设备驱动模型中，需关心总线、设备和驱动这 3 个实体，总线将设备和驱动绑定。

- 在系统每注册一个设备的时候，会寻找与之匹配的驱动；相反的，在系统每注册一个驱动的时候，会寻找与之匹配的设备，而匹配由总线完成。
- 一个现实的 Linux 设备和驱动通常都需要挂接在一种总线上，对于本身依附于 PCI、USB、I2C、SPI 等的设备而言，这自然不是问题，但是在嵌入式系统里面，在 `SoC` 系统中集成的*独立外设控制器*、挂接在 `SoC`*内存空间*的外设等却不依附于此类总线。
- 基于这一背景，`Linux` 发明了一种虚拟的总线，称为 platform 总线，相应的设备称为 `platform_device`，而驱动成为 `platform_driver`。
### 定义
> 实现了同类设备和驱动的**分离**，提高代码的重用性，增强设备驱动的可移植性。

- 平台总线模型也叫`platform`总线模型。是Linux内核**虚拟**出来的一条总线，不是真实的导线。
- 平台总线模型就是把原来的驱动C文件给分成了两个C文件，一个是`device.c`，一个是`driver.c`
- 把稳定不变的放在`driver.c`里面，需要变得就放在了`device.c`里面。
### `Platform`设备
- 在 `platform` 平台下用 `platform_device` 这个结构体表示 `platform` 设备
- 如果内核支持*设备树*的话就不用使用 `platform_device` 来描述设备了，因为改用设备树去描述了` platform_device`
#### struct platform_device
```c
#include<linux/platform_device.h>

/**
 * @brief 描述平台设备结构体
 * 
 * 这个结构体用于定义一个平台设备，它包含设备的一些基本信息。
 * 这些信息通常由平台固件提供，并且被内核在启动时使用。
 */
struct platform_device {
    struct device dev;       /**< 通用设备结构体 */
    struct list_head list;   /**< 用于将设备添加到平台设备列表中的链表 */
    const char *name;        /**< 设备名称 */
    struct resource *resource; /**< 设备使用的资源列表 */
    unsigned int num_resources; /**< 资源列表中的资源数量 */
    void *dev.platform_data;   /**< 特定于设备的平台数据 */
    struct platform_device_info info; /**< 设备信息 */
};
```

#### struct resource
```c
/**
 * @brief 描述资源结构体
 * 
 * 这个结构体用于定义设备所需的物理资源。
 * 它包括资源的起始地址、大小、资源类型等信息。
 */
struct resource {
    resource_size_t start;  /**< 资源的起始地址 */
    resource_size_t end;    /**< 资源的结束地址 */
    unsigned long flags;     /**< 资源的标志，如 IORESOURCE_MEM、IORESOURCE_IO 等,在include<linux/ioport.h>中定义 */
    struct resource *parent; /**< 父资源，用于资源的分组和继承 */
    struct list_head sibling; /**< 用于将资源添加到资源列表中的链表 */
    struct list_head child;   /**< 子资源链表 */
    const char *name;        /**< 资源的名称，用于调试和识别 */
};
```
#### platform_device_register
> 在不支持设备树的 `Linux` 内核版本中需要在通过 `platform_device` 结构体来描述设备信息，然后使用`platform_device_register` 函数将设备信息注册到 `Linux` 内核中

```c
/**
 * @brief 注册一个平台设备
 * 
 * 此函数用于将一个平台设备注册到内核中。注册后，内核会根据设备
 * 的资源和平台数据来初始化设备。
 * 
 * @param pdev 指向平台设备结构体的指针
 * 
 * @return 成功时返回0，失败时返回负错误码
 */
int platform_device_register(struct platform_device *pdev);
```
### Platform 驱动
- 在 Linux 内核中，用 `platform_driver` 结构体表示`platform` 驱动，`platform_driver`结构体定义指定名称的平台设备驱动注册函数和平台设备驱动注销函数
#### struct platform_driver
```c
#include<linux/platform_device.h>

/**
 * @brief 描述平台驱动程序结构体
 * 
 * 这个结构体用于定义一个平台驱动程序，它包含驱动程序的入口点函数，
 * 这些函数用于处理设备的注册、注销、启动和停止等操作。
 */
struct platform_driver {
    int (*probe)(struct platform_device *pdev);    /**< 设备注册时调用的函数 */
    int (*remove)(struct platform_device *pdev);   /**< 设备注销时调用的函数 */
    int (*suspend)(struct platform_device *pdev, pm_message_t state); /**< 设备挂起时调用的函数 */
    int (*resume)(struct platform_device *pdev);    /**< 设备恢复时调用的函数 */
    void (*shutdown)(struct platform_device *pdev); /**< 设备关闭时调用的函数 */
    struct device_driver driver;                   /**< 驱动程序的设备驱动结构体 */
    struct platform_device *devices;               /**< 驱动程序管理的设备列表 */
    struct list_head driver_list;                  /**< 驱动程序链表 */
};
```
#### struct platform_device_id
> `id_table` 表保存了很多 id 信息。

- 这些 id 信息存放着这个 `platform`驱动所支持的*驱动类型*。
- id_table 是个表(也就是数组)，每个元素的类型为 `platform_device_id`
```c
/**
 * @brief 描述平台设备标识符结构体
 * 
 * 这个结构体用于定义平台驱动程序支持的设备标识符。
 * 它包含设备名称和可选的驱动程序私有数据。
 * 当平台设备注册时，内核会使用这些标识符来匹配驱动程序。
 */
struct platform_device_id {
    char name[PLATFORM_DEVICE_ID_SIZE]; /**< 设备名称，用于匹配驱动程序 */
    unsigned long driver_data;         /**< 驱动程序私有数据，用于在匹配时传递额外信息 */
};
```
#### struct of_device_id
> `of_match_table` 就是采用设备树的时候驱动使用的匹配表

- 同样是数组，每个匹配项都为`of_device_id` 结构体类型
- 此结构体定义在文件 `include/linux/mod_devicetable.h`
```c
/**
 * @brief 描述设备树设备标识符结构体
 * 
 * 这个结构体用于定义设备树中设备驱动程序支持的设备标识符。
 * 它包含设备名称、类型、兼容字符串和可选的驱动程序私有数据。
 * 当设备树中的设备节点被解析时，内核会使用这些标识符来匹配驱动程序。
 */
struct of_device_id {
    char name[32];        /**< 设备名称，用于匹配设备树中的设备节点 */
    char type[32];        /**< 设备类型，用于进一步细化设备匹配 */
    char compatible[128];/**< 兼容字符串，用于匹配设备树中的设备节点 */
    const void *data;     /**< 驱动程序私有数据，用于在匹配时传递额外信息 */
};
```
> `compatible` 成员，在支持设备树的内核中，就是通过设备节点的 compatible 属性值和of_match_table 中每个项目的 compatible 成员变量进行比较，如果有相等的就表示设备和此驱动匹配成功。

#### 驱动编写流程
- 在编写 platform 驱动的时候，首先定义一个 platform_driver 结构体变量
- 然后实现结构体中的各个成员变量，重点是实现匹配方法以及 probe 函数。
- 当驱动和设备匹配成功以后 probe 函数就会执行，驱动程序具体功能的实现在 probe 函数里面编写。
#### platform_driver_register
> platform 驱动的注册使用 platform_driver_register 函数来实现
```c
/**
 * @brief 注册一个平台驱动程序
 * 
 * 此函数用于将一个平台驱动程序注册到内核中。注册后，内核会根据设备
 * 的资源和平台数据来初始化设备。
 * 
 * @param driver 指向平台驱动程序结构体的指针
 * 
 * @return 成功时返回0，失败时返回负错误码
 */
int platform_driver_register(struct platform_driver *driver);
```
#### platform_driver_unregister
```c
/**
 * @brief 注销一个平台驱动程序
 * 
 * 此函数用于注销一个已经注册的平台驱动程序。
 * 当驱动程序不再需要管理任何设备时，或者在模块卸载过程中，
 * 应该调用此函数来注销驱动程序，释放相关资源。
 * 
 * @param drv 指向平台驱动程序结构体的指针
 */
void platform_driver_unregister(struct platform_driver *drv);
```
### Platform 总线
> platform 设备和 platform 驱动相当于把设备和驱动分离，使用 platform 总线实现驱动和设备的匹配

#### struct bus_type
> 在 Linux 内核中使用 `bus_type` 结构体表示==总线==
```c
include<linux/device.h>

/*
 * bus_type - 表示一个总线类型的数据结构
 *
 * 这个结构体定义了 Linux 内核中总线类型的属性和方法。
 * 总线类型是内核设备模型的一部分，用于管理连接到系统的设备。
 */
struct bus_type {
    const char *name;                   /* 总线的名字 */
    const char *dev_name;               /* 设备节点的前缀 */
    struct device *dev_root;            /* 该总线下设备树的根节点 */
    struct device_attribute *dev_attrs; /* 该总线特有的设备属性 */

    /*
     * 属性组
     */
    const struct attribute_group **bus_groups; /* 总线相关的属性组 */
    const struct attribute_group **dev_groups; /* 设备相关的属性组 */
    const struct attribute_group **drv_groups; /* 驱动相关的属性组 */

    /*
     * 回调函数
     */
    int (*match)(struct device *dev, struct device_driver *drv); /* 匹配函数 */
    int (*uevent)(struct device *dev, struct kobj_uevent_env *env); /* 发送 uevent 事件 */
    int (*probe)(struct device *dev); /* 探测设备是否存在的函数 */
    int (*remove)(struct device *dev); /* 移除设备的函数 */
    void (*shutdown)(struct device *dev); /* 关闭设备的函数 */

    int (*online)(struct device *dev); /* 设备上线函数 */
    int (*offline)(struct device *dev); /* 设备下线函数 */
    int (*suspend)(struct device *dev, pm_message_t state); /* 挂起设备的函数 */
    int (*resume)(struct device *dev); /* 恢复设备的函数 */

    /*
     * 操作集合
     */
    const struct dev_pm_ops *pm;       /* 设备电源管理操作集合 */
    const struct iommu_ops *iommu_ops; /* IOMMU 操作集合 */

    /*
     * 其他字段
     */
    struct subsys_private *p;          /* 供子系统使用的私有数据 */
    struct lock_class_key lock_key;    /* 用于锁分类的键 */
};
```

> `match` 函数是完成设备和驱动之间的匹配任务，因此每一条总线都必须实现此函数。

- match 函数有两个参数：dev 和 drv，这两个参数分别为 `device` 和 `device_driver` 类型，也就是设备和驱动。
#### platform 总线是 bus_type 的一个具体实例
```c
/*
 * platform_bus_type - 平台总线类型的初始化
 *
 * 这个结构体实例化了平台总线类型，并设置了相应的属性和回调函数。
 * 平台总线类型用于管理通过平台总线连接到系统的设备。
 */
struct bus_type platform_bus_type = {
    .name = "platform",                  /* 总线的名字 */
    .dev_groups = platform_dev_groups,   /* 设备相关的属性组 */
    .match = platform_match,             /* 匹配函数 */
    .uevent = platform_uevent,           /* 发送 uevent 事件的函数 */
    .pm = &platform_dev_pm_ops,          /* 设备电源管理操作集合 */
};
```

#### platform_match
> platform_match 匹配函数，用来匹配注册到 platform 总线的设备和驱动。
```c 
/*
 * platform_match - 平台设备和驱动的匹配函数
 *
 * @dev: 指向设备结构的指针
 * @drv: 指向设备驱动结构的指针
 *
 * 这个函数用于确定一个平台设备是否与给定的平台驱动匹配。
 * 匹配过程依次尝试以下几种方式：
 * 1. 如果设置了 driver_override，则仅当设备的 driver_override 字段与驱动的 name 字段相匹配时才返回成功。
 * 2. 尝试使用 Device Tree 数据进行匹配。
 * 3. 尝试使用 ACPI 数据进行匹配。
 * 4. 尝试与驱动的 ID 表进行匹配。
 * 5. 最后，回退到简单的驱动名称匹配。
 *
 * 返回值：
 * 如果匹配成功，返回 1；否则返回 0。
 */
static int platform_match(struct device *dev, struct device_driver *drv)
{
    struct platform_device *pdev = to_platform_device(dev);
    struct platform_driver *pdrv = to_platform_driver(drv);

    /*
     * 当 driver_override 被设置时，只绑定到匹配的驱动。
     */
    if (pdev->driver_override)
        return !strcmp(pdev->driver_override, drv->name);

    /*
     * 尝试 OF（Device Tree）风格的匹配。
     */
    if (of_driver_match_device(dev, drv))
        return 1;

    /*
     * 然后尝试 ACPI 风格的匹配。
     */
    if (acpi_driver_match_device(dev, drv))
        return 1;

    /*
     * 然后尝试与 ID 表进行匹配。
     */
    if (pdrv->id_table)
        return platform_match_id(pdrv->id_table, pdev) != NULL;

    /*
     * 最后，回退到驱动名称匹配。
     */
    return (strcmp(pdev->name, drv->name) == 0);
}
```
#### probe
> 编写思路
- 从device.c里面获得硬件资源，匹配成功了之后，driver.c要从device.c中获得硬件资源，driver.c就是在probe函数中获得的。
- 获得硬件资源之后，就可以在probe函数中注册杂项/字符设备，完善file_operation结构体，并生成设备节点。
#### 获取硬件资源
> 法一：直接获取--不推荐

- `probe`函数里面有一个形参`pdev`，他指向了`platform_device`
- 我们可以直接通过*指针*访问结构体`platform_device`的*成员变量*。
> 法二：使用函数来获取资源
```c
/*
 * platform_get_resource - 获取平台设备的资源
 *
 * @pdev: 指向平台设备结构的指针
 * @type: 资源类型
 * @num: 资源编号
 *
 * 这个函数用于从给定的平台设备中获取指定类型的资源。
 *
 * 参数：
 * @pdev: 指向平台设备（struct platform_device）的指针。
 * @type: 要查找的资源类型。
 * @num: 要查找的资源编号（同类资源的编号，从0开始）。
 *
 * 返回值：
 * 返回指向请求资源的 struct resource 指针。如果没有找到相应的资源，则返回 NULL。
 */
struct resource *platform_get_resource(struct platform_device *pdev, unsigned int type, unsigned int num)
{
    struct resource *res;
    int i;

    /* 遍历设备的所有资源 */
    for (i = 0; i < pdev->num_resources; i++) {
        res = &pdev->resource[i];

        /* 检查资源类型是否匹配 */
        if (res->flags & type) {
            /* 检查资源编号是否匹配 */
            if (num == 0 || num == i + 1) {
                return res;
            }
        }
    }

    /* 没有找到匹配的资源 */
    return NULL;
}
```
#### 申请`I/O`内存
- inux把基于I/O映射方式的I/O端口和基于内存映射方式的I/O端口资源统称为“I/O区域”（`I/O Region`）。
- ` I/O Region`仍然是一种I/O资源，因此它仍然可以用`resource`结构类型来描述。
- Linux是以一种*倒置*的**树形结构**来管理每一类I/O资源（如：`I/O`端口、外设内存、`DMA`和`IRQ`）。
- 每一类I/O资源都对应有一颗倒置的资源树，树中的每一个节点都是一个`resource`结构，而树的根结点`root`则描述了该类资源的整个**资源空间**
- 换句话说，`request_mem_region`函数并没有做*实际性的映射*工作，只是告诉内核要使用一块内存地址，*声明占有*，也方便内核管理这些资源。
- `#define request_mem_region(start, n, name)  __request_region(&iomem_resource, (start), (n), (name), 0)`
```c
/*
 * request_mem_region - 请求并保留一段内存区域
 *
 * @start: 起始地址
 * @n: 需要保留的大小（以字节为单位）
 * @name: 资源的名称
 *
 * 这个宏用于请求并保留一段内存区域，确保该区域不与其他已保留的区域冲突。
 * 它实际上调用了 __request_region 函数，并且传入了 I/O 内存资源。
 *
 * 参数：
 * @start: 需要保留的内存区域的起始地址。
 * @n: 需要保留的内存区域的大小（以字节为单位）。
 * @name: 资源的名称，用于标识该资源。
 *
 * 返回值：
 * 如果请求成功，返回指向 struct resource 的指针；如果发生错误，则返回 NULL。
 */
#define request_mem_region(start, n, name)  __request_region(&iomem_resource, (start), (n), (name), 0)

/*
 * __request_region - 请求并保留一段内存或 I/O 端口区域
 *
 * @res: 指向资源池的指针
 * @start: 起始地址
 * @n: 需要保留的大小（以字节为单位）
 * @name: 资源的名称
 * @flags: 标志位
 *
 * 这个函数用于请求并保留一段内存或 I/O 端口区域，确保该区域不与其他已保留的区域冲突。
 *
 * 参数：
 * @res: 指向资源池的指针。
 * @start: 需要保留的内存或 I/O 端口区域的起始地址。
 * @n: 需要保留的内存或 I/O 端口区域的大小（以字节为单位）。
 * @name: 资源的名称，用于标识该资源。
 * @flags: 标志位，通常用于指定资源类型或其他标志。
 *
 * 返回值：
 * 如果请求成功，返回指向 struct resource 的指针；如果发生错误，则返回 NULL。
 */
struct resource *__request_region(struct resource *res, unsigned long start, unsigned long n, const char *name, unsigned long flags)
{
    struct resource *new_res;
    struct resource *tmp;
    int ret;

    /* 分配一个新的资源结构 */
    new_res = kzalloc(sizeof(*new_res), GFP_KERNEL);
    if (!new_res) {
        return NULL;
    }

    /* 初始化资源结构 */
    new_res->start = start;
    new_res->end = start + n - 1;
    new_res->flags = flags;
    strncpy(new_res->name, name, sizeof(new_res->name) - 1);
    new_res->name[sizeof(new_res->name) - 1] = '\0';

    /* 检查是否有重叠的资源 */
    list_for_each_entry(tmp, &res->sibling, sibling) {
        if (resource_size(new_res) > 0 &&
            resource_size(tmp) > 0 &&
            resource_busy(new_res, tmp)) {
            kfree(new_res);
            return NULL;
        }
    }

    /* 添加新资源到链表 */
    list_add_tail(&new_res->sibling, &res->sibling);

    return new_res;
}

/* 定义全局资源池 */
struct resource iomem_resource = {
    .name = "iomem",
    .sibling = LIST_HEAD(iomem_resource.sibling),
};
```
## 示例
### 编写device.c
- device.c里面写的是*硬件资源*，这里硬件资源是指寄存器的地址，中断号，时钟等硬件资源。
- 但是硬件资源不是指具体的硬件资源。比如说我要描述led的硬件资源，他描述的不是led灯，他描述的led灯使用了哪些管脚，这些管脚又涉及到了哪些寄存器。
```c
/*
 * @Author: topeet
 * @Description: 基于平台设备模型的device.c
 */
#include <linux/init.h>            //初始化头文件
#include <linux/module.h>          //最基本的文件，支持动态添加和卸载模块。
#include <linux/platform_device.h> //平台设备所需要的头文件
/**
 * @description: 释放 flatform 设备模块的时候此函数会执行
 * @param {structdevice} *dev:要释放的设备
 * @return {*}
 */
void led_release(struct device *dev)
{
    printk("led_release \n");
}
// 设备资源信息，也就是LED所使用的所有寄存器
struct resource led_res[] = {
    [0] = {
        .start = 0x30200000,
        .end = 0x30200003,
        .flags = IORESOURCE_MEM,
        .name = "GPIO1_IO13",
    },
    [1] = {
        .start = 0x30200004,
        .end = 0x30200007,
        .flags = IORESOURCE_MEM,
        .name = "GPIO1_IO13_GDIR",
    },
 
};
// platform 设备结构体
struct platform_device led_device = {
    .name = "led_test",
    .id = -1,
    .resource = led_res,
    .num_resources = ARRAY_SIZE(led_res),
    .dev = {
        .release = led_release}};
/**
 * @description:  设备模块加载
 * @param {*}无
 * @return {*}无
 */
static int device_init(void)
{
    // 设备信息注册到 Linux 内核
    platform_device_register(&led_device);
    printk("platform_device_register ok \n");
    return 0;
}
/**
 * @description: 设备模块注销
 * @param {*}无
 * @return {*}无
 */
static void device_exit(void)
{
    // 设备信息卸载
    platform_device_unregister(&led_device);
    printk("gooodbye! \n");
}
module_init(device_init);
module_exit(device_exit);
MODULE_LICENSE("GPL");
```

- `Makefile`
```c
obj-m += device.o
KDIR:=/home/topeet/linux/linux-imx
PWD?=$(shell pwd)
all:
	make -C $(KDIR) M=$(PWD) modules ARCH=arm64
clean:
	make -C $(KDIR) M=$(PWD) clean
```

> 加载驱动
- 编译完成后，首先使用命令`ls /sys/bus/platform/devices`查看是否存在平台设备`led_test`
- `insmod device.ko`加载驱动
- 再次查看发现有平台设备`led_test`
- 使用`rmmod device`移除驱动

### 编写driver.c
```c
/*
 * @Author:topeet
 * @Description: 基于平台设备模型的driver.c
 */
#include <linux/init.h>            //初始化头文件
#include <linux/module.h>          //最基本的文件，支持动态添加和卸载模块。
#include <linux/platform_device.h> //平台设备所需要的头文件
/**
 * @description: led_probe，驱动和设备匹配成功会进入此函数
 * @param {structplatform_device} *pdev
 * @return {*}
 */
int led_probe(struct platform_device *pdev)
{
    printk("led_probe\n");
    return 0;
}
/**
 * @description: led_remove，当driver和device任意一个remove的时候，就会执行这个函数
 * @param {structplatform_device} *pdev
 * @return {*}
 */
int led_remove(struct platform_device *pdev)
{
    printk("led_remove\n");
    return 0;
}
 
// platform 驱动结构体
struct platform_driver led_driver = {
    .probe = led_probe,
    .remove = led_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "led_test"},
};
/**
 * @description: 设备模块加载
 * @param {*}无
 * @return {*}无
 */
static int led_driver_init(void)
{
    int ret = 0;
    // platform驱动注册到 Linux 内核
    ret = platform_driver_register(&led_driver);
    if (ret < 0)
    {
        printk("platform_driver_register error \n");
    }
    printk("platform_driver_register ok \n");
 
    return 0;
}
/**
 * @description: 设备模块注销
 * @param {*}无
 * @return {*}无
 */
static void led_driver_exit(void)
{
    // platform驱动卸载
    platform_driver_unregister(&led_driver);
    printk("goodbye! \n");
}
module_init(led_driver_init);
module_exit(led_driver_exit);
MODULE_LICENSE("GPL");
```
- Makefile
```make
obj-m += driver.o
KDIR:=/home/topeet/linux/linux-imx
PWD?=$(shell pwd)
all:
	make -C $(KDIR) M=$(PWD) modules ARCH=arm64
clean:
	make -C $(KDIR) M=$(PWD) clean
```
- 将驱动编译为模块
- 使用`insmod driver.ko` `insmod device.ko`加载设备和驱动
- `driver.ko`和`device.ko`的加载顺序没有要求，但是需要`driver.c`中*id_table*中`name`属性设为和`device.c`中的`name`一致
- 预期结果为打印`led_probe`表示设备和驱动匹配成功并进入`probe`函数

>  `platform_driver`结构体中`const struct platform_device_id *id_table`比`device_driver`结构体中的`name`的优先级要高，优先和`id_table`进行匹配
- 修改driver.c
```c
/*
 * @Author:topeet
 * @Description: 基于平台设备模型的driver.c
 */
#include <linux/init.h>            //初始化头文件
#include <linux/module.h>          //最基本的文件，支持动态添加和卸载模块。
#include <linux/platform_device.h> //平台设备所需要的头文件
/**
 * @description: led_probe，驱动和设备匹配成功会进入此函数
 * @param {structplatform_device} *pdev
 * @return {*}
 */
int led_probe(struct platform_device *pdev)
{
    printk("led_probe\n");
    return 0;
}
/**
 * @description: led_remove，当driver和device任意一个remove的时候，就会执行这个函数
 * @param {structplatform_device} *pdev
 * @return {*}
 */
int led_remove(struct platform_device *pdev)
{
    printk("led_remove\n");
    return 0;
}
 
//该设备驱动支持的设备的列表 ，他是通过这个指针去指向 platform_device_id 类型的数组
const struct platform_device_id led_idtable = {
    .name = "123", //设备名字叫“123”
};
 
// platform 驱动结构体
struct platform_driver led_driver = {
    .probe = led_probe,
    .remove = led_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "led_test"
        },
    .id_table=&led_idtable};
/**
 * @description: 设备模块加载
 * @param {*}无
 * @return {*}无
 */
static int led_driver_init(void)
{
    int ret = 0;
    // platform驱动注册到 Linux 内核
    ret = platform_driver_register(&led_driver);
    if (ret < 0)
    {
        printk("platform_driver_register error \n");
    }
    printk("platform_driver_register ok \n");
 
    return 0;
}
/**
 * @description: 设备模块注销
 * @param {*}无
 * @return {*}无
 */
static void led_driver_exit(void)
{
    // platform驱动卸载
    platform_driver_unregister(&led_driver);
    printk("goodbye! \n");
}
module_init(led_driver_init);
module_exit(led_driver_exit);
MODULE_LICENSE("GPL");
```
### 编写probe
```c
/*
 * @Author: topeet
 * @Description: 基于平台设备模型的driver.c,在probe函数中获取硬件资源
 */
//初始化头文件
#include <linux/init.h>
//最基本的文件，支持动态添加和卸载模块。
#include <linux/module.h>
//平台设备所需要的头文件 
#include <linux/platform_device.h>
#include <linux/ioport.h>
/*注册杂项设备头文件*/
#include <linux/miscdevice.h>
//文件系统头文件，定义文件表结构（file,buffer_head,m_inode等）
#include <linux/fs.h>
//包含了copy_to_user、copy_from_user等内核访问用户进程内存地址的函数定义。
#include <linux/uaccess.h>
//包含了ioremap、iowrite等内核访问IO内存等函数的定义。
#include <linux/io.h>
#define GPIO1_IO13 0x30200000      //led数据寄存器物理地址
#define GPIO1_IO13_GDIR 0x30200004 //led数据寄存器方向寄存器物理地址
unsigned int *vir_gpio1_io13;      //存放映射完的虚拟地址的首地址
unsigned int *vir_gpio1_io13_gdir; //存放映射完的虚拟地址的首地址
int ret; //返回值
 
struct resource *led_mem;
struct resource *led_mem_gdir;
struct resource *led_mem_tmp;
 
/****************************************************************************************
 * @brief led_probe : 与设备信息层（device.c）匹配成功后自动执行此函数，
 * @param inode : 文件索引
 * @param file  : 文件
 * @return 成功返回 0           
 ****************************************************************************************/
int led_probe(struct platform_device *pdev)
{
    printk("led_probe\n");
    /*获取硬件资源方法一： 不推荐*/
    //printk("led_res is %s\n",pdev->resource[0].name);
    //return 0;
    /*获取硬件资源方法二： 推荐*/
    led_mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    led_mem_gdir = platform_get_resource(pdev, IORESOURCE_MEM, 1);
    if (led_mem == NULL||led_mem_gdir == NULL)
    {
        printk("platform_get_resource is error\n");
        return -EBUSY;
    }
 
    printk("led_res start is 0x%llx \n", led_mem->start);
    printk("led_res end is 0x%llx \n", led_mem->end);
    printk("led_res start is 0x%llx \n", led_mem_gdir->start);
    printk("led_res end is 0x%llx \n", led_mem_gdir->end);
    return 0;
}
int led_remove(struct platform_device *pdev)
{
    printk("led_remove\n");
    return 0;
}
 
const struct platform_device_id led_idtable = {
    .name = "led_test",
};
// platform 驱动结构体
struct platform_driver led_driver = {
    .probe = led_probe,
    .remove = led_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "123"},
    .id_table = &led_idtable
 
};
/**
 * @description: 设备模块加载
 * @param {*}无
 * @return {*}无
 */
static int led_driver_init(void)
{
    int ret = 0;
    // platform驱动注册到 Linux 内核
    ret = platform_driver_register(&led_driver);
    if (ret < 0)
    {
        printk("platform_driver_register error \n");
    }
    printk("platform_driver_register ok \n");
 
    return 0;
}
/**
 * @description: 设备模块注销
 * @param {*}无
 * @return {*}无
 */
static void led_driver_exit(void)
{
    platform_driver_unregister(&led_driver);
    printk("gooodbye! \n");
}
module_init(led_driver_init);
module_exit(led_driver_exit);
MODULE_LICENSE("GPL");
```
- 将`driver.c`编译为模块
- 预期结果会打印`led_res`对应的地址，表明已获取了硬件资源
- 注册一个杂项设备
```c
/*
 * @Author: topeet
 * @Description: 基于平台设备模型的driver.c,在probe函数中获取硬件资源后，注册一个杂项设备
 */
//初始化头文件
#include <linux/init.h>
//最基本的文件，支持动态添加和卸载模块。
#include <linux/module.h>
//平台设备所需要的头文件 
#include <linux/platform_device.h>
#include <linux/ioport.h>
/*注册杂项设备头文件*/
#include <linux/miscdevice.h>
//文件系统头文件，定义文件表结构（file,buffer_head,m_inode等）
#include <linux/fs.h>
//包含了copy_to_user、copy_from_user等内核访问用户进程内存地址的函数定义。
#include <linux/uaccess.h>
//包含了ioremap、iowrite等内核访问IO内存等函数的定义。
#include <linux/io.h>
#define GPIO1_IO13 0x30200000      //led数据寄存器物理地址
#define GPIO1_IO13_GDIR 0x30200004 //led数据寄存器方向寄存器物理地址
unsigned int *vir_gpio1_io13;      //存放映射完的虚拟地址的首地址
unsigned int *vir_gpio1_io13_gdir; //存放映射完的虚拟地址的首地址
int ret;                           //返回值
 
struct resource *led_mem;
struct resource *led_mem_gdir;
struct resource *led_mem_tmp;
/****************************************************************************************
 * @brief misc_read : 用户空间向设备写入数据时执行此函数
 * @param file  : 文件
 * @param ubuf  : 指向用户空间数据缓冲区
 * @return 成功返回 0       
 ****************************************************************************************/
ssize_t misc_read(struct file *file, char __user *ubuf, size_t size, loff_t *loff_t)
{
    printk("misc_read\n ");
    return 0;
}
/****************************************************************************************
 * @brief misc_write : 用户空间向驱动模块写入数据时执行此函数，对数据进行判断，控制LED亮灭
 * @param file  : 文件
 * @param ubuf  : 指向用户空间数据缓冲区
 * @return 成功返回 0 ，失败返回 -1      
 ****************************************************************************************/
ssize_t misc_write(struct file *file, const char __user *ubuf, size_t size, loff_t *loff_t)
 
{
    char kbuf[64] = {0}; //保存的是从应用层读取到的数据
    if (copy_from_user(kbuf, ubuf, size) != 0)
    {
        printk("copy_from_user error \n ");
        return -1;
    }
    printk("kbuf is %d\n ", kbuf[0]);
    *vir_gpio1_io13_gdir |= (1 << 13);
    if (kbuf[0] == 1) //如果传递进来数据为1，则打开LED
    {
        *vir_gpio1_io13 |= (1 << 13);
    }
    else if (kbuf[0] == 0) //如果传递进来数据为0，关闭LED
        *vir_gpio1_io13 &= ~(1 << 13);
    return 0;
}
/****************************************************************************************
 * @description:misc_release 释放 platform 设备模块的时候此函数会执行
 * @param {structinode} *inode
 * @param {structfile} *file 
 * @return {*}
 ****************************************************************************************/
int misc_release(struct inode *inode, struct file *file)
{
    printk("hello misc_relaease bye bye \n ");
    return 0;
}
/****************************************************************************************
 * @brief misc_open : 打开设备节点时执行此函数，并初始化GPIO
 * @param inode : 文件索引
 * @param file  : 文件
 * @return 成功返回 0           
 ****************************************************************************************/
int misc_open(struct inode *inode, struct file *file)
{
    printk("hello misc_open\n ");
    return 0;
}
 
struct file_operations misc_fops = {
    .owner = THIS_MODULE,
    .open = misc_open,
    .release = misc_release,
    .read = misc_read,
    .write = misc_write,
};
 
struct miscdevice misc_dev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "hello_misc",
    .fops = &misc_fops,
};
/****************************************************************************************
 * @brief led_probe : 与设备信息层（device.c）匹配成功后自动执行此函数，
 * @param inode : 文件索引
 * @param file  : 文件
 * @return 成功返回 0           
 ****************************************************************************************/
int led_probe(struct platform_device *pdev)
{
    printk("led_probe\n");
    /*获取硬件资源方法一： 不推荐*/
    //printk("led_res is %s\n",pdev->resource[0].name);
    //return 0;
    /*获取硬件资源方法二： 推荐*/
    led_mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    led_mem_gdir = platform_get_resource(pdev, IORESOURCE_MEM, 1);
    if (led_mem == NULL || led_mem_gdir == NULL)
    {
        printk("platform_get_resource is error\n");
        return -EBUSY;
    }
 
    printk("led_res start is 0x%llx \n", led_mem->start);
    printk("led_res end is 0x%llx \n", led_mem->end);
    printk("led_res start is 0x%llx \n", led_mem_gdir->start);
    printk("led_res end is 0x%llx \n", led_mem_gdir->end);
 
    /*led_mem_tmp =  request_mem_region(led_mem->start,led_mem->end-led_mem->start+1,"led");
    if(led_mem_tmp == NULL)
    {
        printk("  request_mem_region is error\n");
        goto err_region;
 err_region:
        release_mem_region(led_mem->start,led_mem->end-led_mem->start+1);
    return -EBUSY;
    }*/
    /*****************************************************************/
    //映射GPIO资源
    vir_gpio1_io13 = ioremap(led_mem->start, 4);
    if (vir_gpio1_io13 == NULL)
    {
        printk("GPIO1_IO13 ioremap is error \n");
        return EBUSY;
    }
    printk("GPIO1_IO13 ioremap is ok \n");
 
    vir_gpio1_io13_gdir = ioremap(led_mem_gdir->start, 4);
    if (vir_gpio1_io13_gdir == NULL)
    {
        printk("GPIO1_IO13_GDIR ioremap is error \n");
        return EBUSY;
    }
    printk("GPIO1_IO13_GDIR ioremap is ok \n");
 
    //注册杂项设备
    ret = misc_register(&misc_dev);
    if (ret < 0)
    {
        printk("misc registe is error \n");
    }
    printk("misc registe is succeed \n");
    return 0;
}
int led_remove(struct platform_device *pdev)
{
    printk("led_remove\n");
    return 0;
}
 
const struct platform_device_id led_idtable = {
    .name = "led_test",
};
// platform 驱动结构体
struct platform_driver led_driver = {
    .probe = led_probe,
    .remove = led_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "led_test"},
    .id_table = &led_idtable
 
};
/**
 * @description: 设备模块加载
 * @param {*}无
 * @return {*}无
 */
static int led_driver_init(void)
{
    int ret = 0;
    // platform驱动注册到 Linux 内核
    ret = platform_driver_register(&led_driver);
    if (ret < 0)
    {
        printk("platform_driver_register error \n");
    }
    printk("platform_driver_register ok \n");
 
    return 0;
}
/**
 * @description: 设备模块注销
 * @param {*}无
 * @return {*}无
 */
static void led_driver_exit(void)
{
    platform_driver_unregister(&led_driver);
    misc_deregister(&misc_dev);
    iounmap(vir_gpio1_io13);
    iounmap(vir_gpio1_io13_gdir);
    printk("gooodbye! \n");
}
module_init(led_driver_init);
module_exit(led_driver_exit);
MODULE_LICENSE("GPL");
```
- 首先移除之前的`driver`驱动`rmmod driver`
- 加载新驱动`insmod driver.ko`
- 使用命令`ls /dev/hello_misc`查看是否生成*设备节点*
- 编写`app.c`测试
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
int main(int argc,char *argv[])
{
    int fd;
    char buf[64] = {0};
    fd = open("/dev/hello_misc",O_RDWR);
    if(fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    buf[0] = atoi(argv[1]);//atoi()
    //read(fd,buf,sizeof(buf));
    write(fd,buf,sizeof(buf));
    //printf("buf is %s\n",buf);
    close(fd);
    return 0;
}
```
- `./app 1` `./app 0`运行`app`，可以看到`led`的状态同步变化
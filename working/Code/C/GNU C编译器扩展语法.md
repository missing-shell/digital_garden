
Linux内核中大量使用的GNU C编译器扩展的一些C语言语法。
## C语言标准和编译器

> GCC编译器也对C语言标准做了很多扩展:

- 零长度数组。
- 语句表达式。
- 内建函数。
- `__attribute__`特殊属性声明
- 标号元素。
- case范围。
## 指定初始化
### 指定初始化数组元素

- `GNU C`支持使用`...`表示范围扩展，这个特性不仅可以使用在数组初始化中，也可以使用在`switch-case`语句中。
- `...`和其两端的数据范围`2`和`8`之间也要有**空格**
### 指定初始化结构体成员

- 指定初始化不仅使用灵活，而且还有一个好处，就是代码易于*维护*。

## 宏构造“利器”：语句表达式

### 语句表达式

- `GNU C`对`C`语言标准作了扩展，允许在一个表达式里内嵌语句，允许在表达式内部使用局部变量、`for`循环和`goto`跳转语句。
```c
({表达式1; 表达式2; 表达式3;})
```
- 语句表达式最外面使用小括号`()`括起来，里面一对大括号`{}`包起来的是代码块，代码块里允许内嵌各种语句
- 语句表达式的值为内嵌语句中*最后一个表达式*的值。
### 在宏定义中使用语句表达式
`typeof`是`GNU C`新增的一个关键字，用来**获取数据类型**。
#### 比较大小宏
1. 良好-可解决运算符优先级问题
```c
#define MAX(x,y)  ((x)>(y) ? (x) : (y))
```

2. 优秀-解决变量自增问题
```c
#define MAX(x,y) ({ \
    int _x = x; \
    int _y = y; \
    _x > _y ? _x : _y; \
})
```

3. `typeof`
```c
#define max(x, y) ({ \
    typeof(x) _x = (x); \
    typeof(y) _y = (y); \
    (void)(&_x == &_y); \
    _x > _y ? _x : _y; \
})
```
### typeof关键字
- 除了使用`typeof`获取基本数据类型，`typeof`还有其他一些高级的用法。
```c
typeof (int *) y;       // 把 y 定义为指向 int 类型的指针，相当于 int *y;
typeof (int) *y;        // 定义一个执行 int 类型的指针变量 y，相当于 int *y;
typeof (*x) y;          // 定义一个指针 x 所指向类型的指针变量 y，相当于 typeof(*x) *y;
typeof (int) y[4];      // 相当于定义一个 int y[4];
typeof (*x) y[4];       // 把 y 定义为指针 x 指向的数据类型的数组，相当于 typeof(*x) y[4];
typeof (typeof (char *)[4]) y; // 相当于定义字符指针数组 char *y[4];
typeof (int x[4]) y;    // 相当于定义 int y[4];
```
### Linux内核中的container_of宏
- 根据结构体某一成员的地址，获取这个**结构体的首地址**。
- 也就是说，如果我们知道了一个结构体的类型和结构体内某一成员的地址，就可以获得这个结构体的首地址。
```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)

#define container_of(ptr, type, member) ({ \
    const typeof(((type *)0)->member) *__mptr = (ptr); \
    (type *)((char *)__mptr - offsetof(type, member)); \
})
```
- 根据每个成员的数据类型和字节对齐方式，编译器会按照结构体中各个成员的顺序，在内存中分配一片连续的空间来存储它们。
- 用结构体成员的地址，减去该成员在结构体内的**偏移**，就可以得到该结构体的首地址了。
## 零长度数组
- 零长度数组有一个奇特的地方，就是它**不占用内存**存储空间。
### 变长结构体
- 零长度数组一般单独使用的机会很少，它常常作为结构体的一个成员，构成一个*变长结构体*。
- 在一个变长结构体中，零长度数组不占用结构体的存储空间，但是我们可以通过使用结构体的成员去访问内存，非常方便。
- 在运行时动态分配内存，需确保按时释放内存。

变长结构体的使用示例
```c
/**
 * @brief A structure representing a dynamic buffer with a variable length array.
 *
 * This structure is designed to hold an integer length followed by a variable number of integers.
 * The actual buffer size is determined at runtime.
 */
struct buffer {
    int len;   /**< The length of the array */
    int a[0];  /**< A flexible array member, which is an array of integers */
};

/**
 * @brief The main function to demonstrate the usage of the buffer structure.
 *
 * @return int Returns 0 on successful execution.
 */
int main(void) {
    struct buffer *buf;  /**< A pointer to a buffer structure */

    // Allocate memory for the buffer structure and an additional 20 integers
    buf = (struct buffer *)malloc(sizeof(struct buffer) + 20 * sizeof(int));
    if (buf == NULL) {
        // Handle memory allocation failure
        return -1;
    }

    buf->len = 20;  /**< Set the length of the array to 20 */

    // Copy the string "hello zhaixue.cc!\n" into the array, treating it as an array of characters
    strcpy((char *)buf->a, "hello zhaixue.cc!\n");

    // Print the string stored in the array
    puts((char *)buf->a);

    // Free the allocated memory
    free(buf);

    return 0;  /**< Return 0 to indicate successful execution */
}
```
- 在这个程序中，我们使用`malloc`申请一片内存，大小为`sizeof(buffer)+20`，即`24`字节。其中`4`字节用来表示内存的长度`20`，剩下的`20`字节空间，才是我们真正可以使用的内存空间
- 我们可以通过结构体成员a直接访问这片内存。

### 内核中的零长度数组
`urb` 的结构体，在Linux内核中用于表示一个USB请求块（USB Request Block）。这个结构体包含了许多字段，用于管理USB数据传输的各种参数和状态。
```c
/**
 * @brief A structure representing a USB Request Block (URB).
 *
 * This structure is used to manage USB data transfers and contains various fields
 * for parameters and status of the transfer.
 */
struct urb {
    struct kref kref;          /**< A reference counter for the URB */
    void *hcpriv;              /**< Private data for the host controller */
    atomic_t use_count;        /**< The number of times the URB is in use */
    atomic_t reject;           /**< The number of times the URB has been rejected */
    int unlinked;              /**< Flag indicating whether the URB has been unlinked */
    struct list_head urb_list; /**< A list head for linking URBs */
    struct list_head anchor_list; /**< A list head for anchoring URBs */
    struct usb_anchor *anchor; /**< Pointer to the USB anchor */
    struct usb_device *dev;    /**< Pointer to the USB device */
    struct usb_host_endpoint *ep; /**< Pointer to the USB host endpoint */
    unsigned int pipe;         /**< The USB pipe identifier */
    unsigned int stream_id;    /**< The stream identifier for the URB */
    int status;                /**< The status of the URB */
    unsigned int transfer_flags; /**< Flags for the data transfer */
    void *transfer_buffer;     /**< Pointer to the data buffer for the transfer */
    dma_addr_t transfer_dma;   /**< The DMA address of the transfer buffer */
    struct scatterlist *sg;    /**< Pointer to the scatter-gather list */
    int num_mapped_sgs;        /**< The number of mapped scatter-gather entries */
    int num_sgs;               /**< The total number of scatter-gather entries */
    u32 transfer_buffer_length; /**< The length of the transfer buffer */
    u32 actual_length;         /**< The actual length of data transferred */
    unsigned char *setup_packet; /**< Pointer to the setup packet for control transfers */
    dma_addr_t setup_dma;      /**< The DMA address of the setup packet */
    int start_frame;           /**< The starting frame for isochronous transfers */
    int number_of_packets;     /**< The number of packets for isochronous transfers */
    int interval;              /**< The interval between packets for isochronous transfers */
    int error_count;           /**< The number of errors that occurred during the transfer */
    void *context;             /**< A pointer to context-specific data */
    usb_complete_t complete;   /**< A callback function for when the transfer is complete */
    struct usb_iso_packet_descriptor iso_frame_desc[0]; /**< An array of isochronous packet descriptors */
};
```
- **kref**: 用于引用计数，帮助管理URB的生命周期。
- **hcpriv**: 为特定主机控制器保留的私有数据。
- **use_count**: 当前URB被引用的次数。
- **reject**: URB被拒绝的次数。
- **unlinked**: 表示URB是否已被取消。
- **urb_list** 和 **anchor_list**: 用于将URB链接到列表中，以便管理和同步。
- **anchor**: 指向URB所属的USB锚点。
- **dev** 和 **ep**: 分别指向URB所属的USB设备和端点。
- **pipe**: 表示URB使用的USB管道。
- **stream_id**: 对于流式传输，表示流的标识符。
- **status**: URB的当前状态。
- **transfer_flags**: 传输的标志，用于控制传输行为。
- **transfer_buffer** 和 **transfer_dma**: 指向传输数据的缓冲区及其DMA地址。
- **sg**: 指向散列表的指针，用于描述数据的物理布局。
- **num_mapped_sgs** 和 **num_sgs**: 分别表示已映射和总的散列项数。
- **transfer_buffer_length** 和 **actual_length**: 分别表示传输缓冲区的长度和实际传输的数据长度。
- **setup_packet** 和 **setup_dma**: 对于控制传输，指向设置包及其DMA地址。
- **start_frame**, **number_of_packets**, **interval**: 对于等时传输，表示开始帧、数据包数量和包间隔。
- **error_count**: 传输过程中发生的错误计数。
- **context**: 一个指向与URB相关的上下文数据的指针。
- **complete**: 一个回调函数，当传输完成时被调用。
- **iso_frame_desc**: 一个数组，包含等时传输的每个数据包的描述符。

### 指针与零长度数组
> 为什么不使用指针来代替零长度数组？

- 数组名用来表征一块连续内存空间的地址，而指针是一个变量，编译器要给它单独分配一个内存空间，用来存放它指向的变量的地址
## 属性声明：section

### GNU C编译器扩展关键字：attribute
> `__attribute__`关键字用来声明一个函数、变量或类型的**特殊属性**。

- `__attribute__`后面是两对小括号，不能图方便只写一对，否则编译就会报错。
- 多个属性都放在`__attribute__（()）`的两对小括号里面，属性之间用逗号隔开。如果属性有自己的参数，则属性的参数同样要用小括号括起来。
- 同时属性声明要紧挨着**变量**。
### 属性声明：section
> 在程序编译时，将一个函数或变量放到指定的段，即放到指定的`section`中。

- `readelf`命令，就可以查看这个可执行文件中各个`section`的基本信息，如大小、起始地址等
- 编译器在编译程序时，以源文件为单位，将一个个源文件编译生成一个个*目标文件*。在编译过程中，编译器都会按照这个默认规则，将函数、变量分别放在不同的`section`中，最后将各个`section`组成一个目标文件
- 编译过程结束后，链接器会将各个目标文件组装合并、重定位，生成一个*可执行文件*。
### U-boot镜像自复制分析
U-boot一般存储在NOR Flash或NAND Flash上。无论从NOR Flash还是从NAND Flash启动，U-boot其本身在启动过程中，都会从Flash存储介质上*加载*自身代码到内存，然后进行*重定位*，跳到内存RAM中去执行。

```c
char __image_copy_start[0] __attribute__((section(".__image_copy_start")));
char __image_copy_end[0] __attribute__((section(".__image_copy_end")));
```
分别定义一个零长度数组，并指示编译器要分别放在`.__image_copy_start`和`.__image_copy_end`这两个`section`中。

> U-boot的链接脚本U-boot.lds
```ld
OUTPUT_FORMAT("elf32-littlearm","elf32-littlearm","elf32-littlearm")

OUTPUT_ARCH(arm)          // 指定目标架构为ARM
ENTRY(_start)            // 指定程序的入口点为 _start 符号

SECTIONS                 // 开始定义段的布局
{
    . = 0x00000000;      // 设置当前地址为0x00000000，通常是内存的起始地址
    . = ALIGN(4);        // 将当前地址对齐到4字节边界

    .text :              // 定义文本段，包含程序的代码
    {
        *(._image_copy_start) // 包含从 __image_copy_start 到 __image_copy_end 的代码
        *(.vectors)           // 包含中断向量表
        arch/arm/cpu/armv7/start.o(.text*) // 包含特定于ARMv7架构的启动代码
        *(.text*)             // 包含所有其他文本段
    }
    . = ALIGN(4);        // 再次对齐到4字节边界

    .data :              // 定义数据段，包含已初始化的全局变量
    {
        *(.data*)           // 包含所有数据段
    }

    .image_copy_end :    // 标记代码复制部分的结束
    {
        *(._image_copy_end) // 包含 __image_copy_end 标记
    }

    .end :               // 定义程序结束段
    {
        *(.end)            // 包含程序结束标记
    }

    _image_binary_end = .; // 定义一个符号，表示二进制文件的结束位置
    . = ALIGN(4096);      // 将当前地址对齐到4096字节边界，通常用于页对齐

    .mmutable :           // 定义一个段，包含可变数据
    {
        *(.mmutable)       // 包含所有可变数据段
    }

    .bss_start_rel_dyn_start (OVERLAY) : // 定义BSS段的起始地址，可以与其他段重叠
    {
        KEEP(*(.bss_start)); // 保留BSS段的开始部分
        _bss_base = .;       // 定义BSS段的基地址
    }

    .bss_bss_base (OVERLAY) : // 定义BSS段，包含未初始化的全局变量
    {
        *(.bss*)            // 包含所有BSS段
        . = ALIGN(4);       // 对齐到4字节边界
        _bss_limit = .;      // 定义BSS段的限制地址
    }

    .bss_end_bss_limit (OVERLAY) : // 定义BSS段的结束地址，可以与其他段重叠
    {
        KEEP(*(.__bss_end)); // 保留BSS段的结束部分
    }
}
```
`__image_copy_start`和`__image_copy_end`这两个`section`，在链接的时候分别放在了代码段`.text`的前面、数据段`.data`的后面，作为`U-boot`复制自身代码的*起始地址*和*结束地址*

> 在`arch/arm/lib/relocate.S`中，`ENTRY`（`relocate_code`）汇编代码主要完成代码复制的功能
```assembly
ENTRY(relocate_code)
    ldr r1, =_image_copy_start    /* r1 <- SRC &_image_copy_start */
    subs r4, r0, r1                /* r4 <- relocation offset */
    beq relocate_done             /* skip relocation if r0 == r1 */
    ldr r2, =_image_copy_end      /* r2 <- SRC &_image_copy_end */

copy_loop:
    ldmia r1!, {r10-r11}           /* copy from source address [r1] */
    stmia r0!, {r10-r11}           /* copy to target address [r0] */
    cmp r1, r2                     /* compare source address with end address [r2] */
    blo copy_loop                  /* branch if less, continue copying */

relocate_done:
    /* Relocation done, rest of the code */
```
- 寄存器`R1`、`R2`分别表示要复制镜像的起始地址和结束地址，`R0`表示要复制到`RAM`中的地址，`R4`存放的是源地址和目的地址之间的偏移
- 通过`ARM`的`LDR`伪指令，直接获取要复制镜像的首地址，并保存在`R1`寄存器中

## 属性声明：aligned

### 地址对齐：aligned
`GNU C`通过`__attribute__`来声明`aligned`和`packed`属性，指定一个变量或类型的**对齐方式**。

既然地址对齐会造成一定的内存空洞，那么我们为什么还要按照这种对齐方式去存储数据呢？一个主要原因就是：这种对齐设置可以简化`CPU`和内存`RAM`之间的接口和硬件设计。
### 结构体的对齐
- 结构体作为一种复合数据类型，编译器在给一个结构体变量分配存储空间时，不仅要考虑结构体内各个基本成员的地址对齐，还要考虑结构体*整体*的对齐。
- 根据结构体的对齐规则，结构体的整体对齐要按结构体所有成员中最大对齐字节数或其整数倍对齐，或者说结构体的整体长度要为其最大成员字节数的整数倍，如果不是整数倍则要补齐。
- 整个结构体的对齐只要按**最大成员**的*对齐字节数*对齐即可
- 通过aligned属性，我们可以显式指定一个变量的对齐方式，但编译器不一定会按照我们指定的大小对齐。
### 属性声明：packed
aligned属性一般用来增大变量的地址对齐，元素之间因为地址对齐会造成一定的内存空洞。而packed属性则与之相反，一般用来*减少地址对齐*，指定变量或类型使用最可能小的地址对齐方式。

在ARM芯片中，每一个控制器的*寄存器地址空间*一般都是连续存在的。如果考虑数据对齐，则结构体内就可能有空洞，就和实际连续的寄存器地址不一致。使用packed可以避免这个问题，结构体的每个成员都紧挨着，依次分配存储地址，这样就避免了各个成员因地址对齐而造成的内存空洞。
### 内核中的aligned、packed声明
> 在Linux内核源码中，我们经常看到`aligned`和`packed`一起使用

- 既避免了结构体内各成员因地址对齐产生内存空洞，又指定了整个结构体的对齐方式

## 属性声明：format

### 变参函数的格式检查
> `GNU`通过`__attribute__`扩展的format属性，来指定变参函数的**参数格式检查**。

```c
__attribute__((format(archetype, string-index, first-to-check)))
void LOG(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
```
### 变参函数的设计与实现
> 变参函数，顾名思义，和`printf()`函数一样，其参数的个数、类型都不固定。

首先解析实际传进来的实参，保存起来，然后才能像普通函数那样，对实参进行各种操作。

> 涉及指针运算，一定要注意，因为每一个参数的地址都是`4`字节大小，所以我们获取下一个参数的地址是`（char*）&count+4`；

不同类型的指针加`1`操作，转换为实际的数值运算是不一样的：
- 对于一个指向int类型的指针变量`p`，`p+1`表示`p+1*sizeof(int)`，对于一个指向`char`类型的指针变量，`p+1`表示`p+1*sizeof(char)`。

编译器提供的相关宏定义：
- `va_list`：定义在编译器头文件`stdarg.h`中，如`typedef char*va_list`；。
- `va_start(fmt，args)`：根据参数`args`的地址，获取`args`后面参数的地址，并保存在`fmt`指针变量中。
- `va_end(args)`：释放`args`指针，将其赋值为`NULL`。

`vprintf()`函数的声明在`stdio.h`头文件中：

```c
// CRTIMP 是一个宏，用于指定函数是 C 运行时库的一部分
// __cdecl 是调用约定，表示函数参数从右到左弹出
// __MINGW_NOTHROW 表示这个函数不会抛出异常
CRTIMP int __cdecl __MINGW_NOTHROW 

// 格式化输出函数，使用可变参数列表
//vprintf()函数有两个参数：一个是格式字符串指针，一个是变参列表。
vprintf(const char* format, __VALIST arglist); 
```
## 属性声明：weak

### 强符号和弱符号
> `GNU C`通过`weak`属性声明，可以将一个强符号转换为弱符号

```c
void __attribute__((weak)) func(void);
int num __attribute__((weak));
```
- *强符号*：函数名，初始化的全局变量名。
- *弱符号*：未初始化的全局变量名。

强符号和弱符号主要用来解决在程序**链接**过程中，出现多个同名全局变量、同名函数的冲突问题：
- 一山不容二虎。
- 强弱可以共处。
- 体积大者胜出。
### 函数的强符号与弱符号
链接器对于同名函数冲突，同样遵循相同的规则。函数名本身就是一个强符号，在一个工程中定义两个同名的函数，编译时肯定会报重定义错误。

### 弱符号的用途
> 当函数被声明为一个弱符号时

- 当*链接器*找不到这个函数的定义时，也不会报错。
- 只有当程序**运行时**，调用到这个函数，跳转到零地址或一个特殊的地址才会报错，产生一个内存错误。
- 为了防止函数运行出错，我们可以在运行这个函数之前，先进行判断，看这个函数名的地址是不是`0`，然后决定是否调用和运行，这样就可以避免**段错误**了。
- 函数名的本质就是一个地址，在调用`func()`之前，我们先判断其是否为`0`，如果为`0`，则不调用，直接跳过。
### 属性声明：alias
> `GNU C`扩展了一个`alias`属性，主要用来给函数定义一个**别名**。

在`Linux`内核中，你会发现`alias`有时会和`weak`属性一起使用。如有些函数随着内核版本升级，函数接口发生了变化，我们可以通过`alias`属性对这个旧的接口名字进行封装，重新起一个接口名字。
## 内联函数

### 属性声明：noinline
> 与内联函数相关的两个属性：`noinline`和`always_inline`。
```c
static inline __attribute__((noinline)) int func();
static inline __attribute__((always_inline)) int func();
```
### 什么是内联函数
一个使用`inline`声明的函数被称为内联函数，内联函数一般前面会有`static`和`extern`修饰。

使用`inline`声明一个内联函数，和使用关键字`register`声明一个寄存器变量一样，只是建议编译器在编译时内联展开。

一个函数在执行过程中，如果需要调用其他函数，则一般会执行下面的过程:
1. 保存当前函数现场。
2. 跳到调用函数执行。
3. 恢复当前函数现场。
4. 继续执行当前函数。

编译器在编译过程中遇到内联函数，像宏一样，将内联函数直接在调用处展开，这样做就减少了函数调用的开销：直接执行内联函数展开的代码，不用再保存现场和恢复现场。

> 与宏相比，内联函数有以下优势：

- *参数类型检查*。
- *便于调试*：函数支持的调试功能有断点、单步等，内联函数同样支持。
- *返回值*：内联函数有返回值，返回一个结果给调用者。这个优势是相对于`ANSI C`说的，因为现在宏也可以有返回值和类型了，如上文使用语句表达式定义的宏。
- *接口封装*：有些内联函数可以用来封装一个接口，而宏不具备这个特性
### 编译器对内联函数的处理

> 内联函数缺点

- 内联函数会增大程序的体积，如果在一个文件中多次调用内联函数，多次展开，那么整个程序的体积就会变大，在一定程度上会降低程序的执行效率
- 内联函数往往又降低了函数的复用性

判断对一个内联函数是否做展开
- 函数体积小。
- 函数体内无指针赋值、递归、循环等语句。
- 调用频繁。
### 内联函数为什么定义在头文件中

> 内联函数为什么要定义在头文件中呢？

因为它是一个内联函数，可以像宏一样使用，任何想使用这个内联函数的源文件，都不必亲自再去定义一遍，直接包含这个头文件，即可像宏一样使用。

> 为什么还要用`static`修饰呢？

- 使用`inline`定义的内联函数，编译器**不一定会内联展开**，那么当一个工程中多个文件都包含这个内联函数的定义时，编译时就有可能报*重定义错误*。
- 使用`static`关键字修饰，则可以将这个函数的作用域限制在各自的文件内，避免重定义错误的发生。

## 内建函数

### 什么是内建函数
内建函数，顾名思义，就是编译器内部实现的函数。这些函数和关键字一样，可以直接调用，无须像标准库函数那样，要先声明后使用。

内建函数的函数命名，通常以`__builtin`开头。这些函数主要在*编译器内部*使用，主要是为编译器服务。
- 处理变长参数列表。
- 处理程序运行异常、编译优化、性能优化。
- 查看函数运行时的底层信息、堆栈信息等。
- 实现`C`标准库的常用函数。

### 常用的内建函数

常用的内建函数主要有两个：
- `__builtin_return_address()`
- `__builtin_frame_address()`

```C
__builtin_return_address(LEVEL)
```
返回当前函数或调用者的返回地址。
函数的参数`LEVEL`表示函数调用链中不同层级的函数。
- `0`：获取当前函数的返回地址。
- `1`：获取上一级函数的返回地址。
- `2`：获取上二级函数的返回地址。
- 每一层函数调用，都会将当前函数的下一条指令地址，即返回地址压入堆栈保存，各级函数调用就构成了一个函数调用链。
- 在函数调用过程中，还有一个栈帧的概念。函数每调用一次，都会将当前函数的现场（返回地址、寄存器、临时变量等）保存在栈中，每一层函数调用都会将各自的现场信息保存在各自的栈中。这个栈就是当前函数的栈帧
- 每个栈帧都会保存上一层栈帧的起始地址，这样各个栈帧就形成了一个调用链。
### C标准库的内建函数

在`GNU C`编译器内部，`C`标准库的内建函数实现了一些与`C`标准库函数类似的内建函数。这些函数与`C`标准库函数功能相似，函数名也相同，只是在前面加了一个前缀`__builtin`。

### 内建函数：`__builtin_constant_p(n)`

主要用来判断参数n在编译时是否为**常量**。如果是常量，则函数返回`1`，否则函数返回`0`。该函数常用于宏定义中，用来编译优化。
### 内建函数：`__builtin_expect(exp，c)`

- 有`2`个参数，返回值就是其中一个参数，仍是`exp`。
- 主要是告诉编译器：参数`exp`的值为`c`的可能性很大，然后编译器可以根据这个提示信息，做一些*分支预测*上的代码优化。
- 参数`c`与这个函数的返回值无关，无论`c`为何值，函数的返回值都是`exp`。
### Linux内核中的likely和unlikely
`if/switch`这种选择分支的程序结构，一般建议将大概率发生的分支写在前面。当程序运行时，因为大概率发生，所以大部分时间就*不需要跳转*，程序就相当于一个顺序结构，`Cache`的缓存命中率也会大大提升。

> 在`Linux`内核中，我们使用`__builtin_expect()`内建函数，定义了两个宏。

```c
#define likely(x) __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x)
```
对宏的参数`x`做**两次取非**操作，这是为了将参数`x`转换为`bool`类型，然后与`1`和`0`直接做比较，告诉编译器x为真或假的可能性很高。
## 可变参数宏
### 什么是可变参数宏
可变参数宏的实现形式其实和变参函数差不多：用`...`表示变参列表，变参列表由不确定的参数组成，各个参数之间用逗号隔开。
```c
#define LOG(fmt, ...) printf(fmt, ##__VA_ARGS__)
```

可变参数宏使用`C99`标准新增加的一个`__VA_ARGS__`*预定义标识符*来表示前面的变参列表，而不是像变参函数一样，使用va_list、va_start、va_end这些宏去解析变参列表。预处理器在将宏展开时，会用变参列表替换掉宏定义中的所有`__VA_ARGS__`标识符。

> 宏连接符`##`的主要作用就是连接两个字符串

在宏定义中可以使用`##`来连接两个字符，预处理器在预处理阶段对宏展开时，会将`##`两边的字符合并，并删除`##`这个连接符。

> 在可变参数宏中使用`##`的目的：
- 连接`fmt`和变参列表，各个参数之间用逗号隔开。
- 当变参列表为空时，`##`还有一个特殊的用处，它会将固定参数fmt后面的逗号删除掉。
### 可变参数宏的另一种写法
使用预定义标识符`__VA_ARGS__`来定义一个变参宏，是`C99`标准规定的写法。

`GNU C`扩展的一个新写法：可以不使用`__VA_ARGS__`，而是直接使用`args...`来表示一个变参列表，然后在后面的宏定义中，直接使用`args`代表变参列表
```c
#define LOG(fmt, args...) printf(fmt, ##args)

int main(void) {
    LOG("hello\n");
    return 0;
}
```

### 内核中的可变参数宏
> 可变参数宏在内核中主要用于日志打印。

> 宏定义采用`do{...}while(0)`结构的好处：
- 防止宏在条件、选择等分支结构的语句中展开后，产生宏歧义。
- 采用`do{...}while(0)`这种结构，可以将我们宏定义中的复合语句包起来。宏展开后，是一个*代码块*，避免了这种逻辑错误。
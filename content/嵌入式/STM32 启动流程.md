## STM32 F103
STM32F103 是基于 [[STM32 启动流程#ARM Cortex-M3|ARM Cortex-M3]] 内核的微控制器。
### 存储器
程序存储器、数据存储器、寄存器和输入输出端口被组织在同一个`4GB`的[[STM32 启动流程#ARM地址分布|线性地址空间内]]。 数据字节以**小端格式**存放在存储器中。

可访问的存储器空间被分成8个主要块，每个块为`512MB`。

`STM32F10xxx`内置`64K`字节的静态`SRAM`。它可以以字节、半字(`16`位)或全字(`32`位)访问。 `SRAM`的起始地址是`0x2000 0000`。

高达512K字节闪存存储器结构：**闪存存储器**有主存储块和信息块组成： 
- 主存储块容量： 
	小容量产品主存储块最大为4K×64位，每个存储块划分为32个1K字节的页。 
	中容量产品主存储块最大为16K×64位，每个存储块划分为128个1K字节的页。 
	大容量产品主存储块最大为64K×64位，每个存储块划分为256个2K字节的页。 
	互联型产品主存储块最大为32K×64位，每个存储块划分为128个2K字节的页。 

- 信息块容量： 
	互联型产品有2360×64位。 
	其它产品有258×64位。
#### Memory Map
![[STM32 Memory map.png]]
### 启动配置
![[STM32F10XXX启动模式.png]]
- Boot from **main Flash memory**: the main Flash memory is aliased in the boot memory space (`0x0000 0000`), but still accessible from its original memory space (`0x800 0000`). In other words, the Flash memory contents can be accessed starting from address 0x0000 0000 or 0x800 0000. 
- Boot from **system memory**: the system memory is aliased in the boot memory space (0x0000 0000), but still accessible from its original memory space (`0x1FFF B000` in connectivity line devices, 0x1FFF F000 in other devices). 
- Boot from **the embedded SRAM**: SRAM is accessible only at address `0x2000 0000`.
When booting from SRAM, in the application initialization code, you have to `relocate `the vector table in SRAM using the `NVIC` exception table and offset register.
### STM32F103 系列的特点

1. **片上资源：**
    - Flash 容量范围：16 KB 到 1 MB。
    - SRAM 容量范围：6 KB 到 96 KB。
2. **丰富的外设接口：**
    - 支持 UART、SPI、I2C、CAN 等多种通信接口。
    - 集成多达 12 个通道的 12 位 ADC。
3. **低功耗特性：**
    - 提供多种低功耗模式（如睡眠模式、停止模式、待机模式）。
### **应用场景：**

STM32F103 常用于实时控制、工业自动化、消费电子等嵌入式领域，兼具性能与成本优势。
### 可执行文件组成

| 程序组件               | 所属类别         |
|------------------------|------------------|
| 机器代码指令           | Code             |
| 常量                   | RO-data（只读数据） |
| 初值非0的全局变量      | RW-data（可读写数据） |
| 初值为0的全局变量      | ZI-data（零初始化数据） |
| 局部变量               | ZI-data 栈空间   |
| 使用malloc动态分配的空间 | ZI-data 堆空间   |
### Bootloader 作用
在嵌入式系统中，**Bootloader** 是系统启动的关键部分，它主要承担系统启动过程中初始化硬件、加载主程序等功能。 
#### 硬件初始化

- **上电后硬件状态未定义：**
    - 上电后，存储器（如 SRAM）未初始化，外设未配置。
    - CPU 的寄存器值和系统状态可能不适合直接运行用户代码。
- **Bootloader 负责硬件初始化：**
    - 配置时钟（如 PLL），提高主频。
    - 初始化外设（如 UART、GPIO）。
    - 设置系统堆栈和中断向量表。
#### 程序加载与升级

- **加载主程序：**
    - Bootloader 从固定存储设备（如 Flash、SD 卡、外部 EEPROM）读取主程序。
    - 将主程序加载到 RAM 中（如果需要）。
- **支持固件更新：**
    - 提供稳定的固件更新机制（如通过串口或网络），便于远程维护。
    - 具备错误校验功能，确保主程序的正确性。
#### 多功能引导需求

- 支持多种启动模式：例如，启动不同的操作系统或功能模块。
- 在某些设备中，Bootloader 可以切换到诊断模式或工厂测试模式。
#### 增强安全性

- 通过加密与验证确保主程序的完整性和合法性，防止恶意篡改或非法代码执行。
### 为什么不直接运行主程序？

- **硬件状态不确定**：主程序需要依赖特定的硬件初始化环境，例如时钟、堆栈配置等。如果直接运行主程序，很可能由于硬件未正确初始化导致运行失败。
* **无法支持动态功能**：Bootloader 提供了一种灵活的机制，允许系统根据需要动态选择启动模式、支持固件更新或切换功能模式。
- **安全需求**：通过 Bootloader，可以在启动过程中验证固件的合法性，防止恶意软件加载。
- **维护性**：嵌入式设备需要在现场维护时具备安全可靠的更新能力。Bootloader 是实现此功能的核心。
## ARM Cortex-M3
1. **ARM Cortex-M3** 是一款 32 位处理器内核，专为嵌入式系统设计，具有高效能和低功耗的特点。
2. 它是 ARMv7-M 架构的实现，支持**哈佛结构**（分离的指令和数据存储）。
3. **工作频率：** STM32F103 系列的主频最高可达 72 MHz。
### ARM启动流程
通用 ARM 启动流程可以简化为以下步骤：

1. **硬件复位**：处理器从复位地址加载栈指针和复位向量。
2. **复位处理**：初始化系统资源，准备运行环境。
3. **执行主程序**：进入用户代码的 `main()` 函数。
4. **中断处理**：通过 NVIC 管理外部中断和系统异常。
### ARM Cortex-M3 的主要特性
三级流水线：取值、解码、执行

1. **指令集支持：**
    - 支持 **Thumb-2 指令集**，相比传统 ARM 指令集，其代码密度更高，执行效率更高。
    - 不支持传统的 ARM 32 位指令集（ARM 模式）。
2. **NVIC（嵌套向量中断控制器）：**
    - 支持多达 256 个中断向量。
    - 提供优先级分组和动态优先级管理。
3. **片上调试：**
    - 内置调试功能，如 **JTAG** 和 **SWD（串行调试接口）**，便于嵌入式开发和调试。
4. **存储器保护单元（MPU）：**
    - 提供基础的内存访问保护功能，有助于提高系统可靠性。
### ARM地址分布
| 地址范围         | 类型                  | 描述                                          |
| ------------ | ------------------- | ------------------------------------------- |
| `0x00000000` | 起始区域                | 默认映射到 Flash，也可通过重映射指向 SRAM 或 Bootloader 区域。 |
| `0x08000000` | 内部 Flash            | 程序代码存储区，大小依芯片型号而定。                          |
| `0x1FFF0000` | 系统存储（System Memory） | STM32 系列中包含 Bootloader 等系统固件。               |
| `0x20000000` | 内部 SRAM             | 用于运行时的变量、栈和堆。                               |
| `0x40000000` | 外设寄存器区域             | 包括 GPIO、UART、SPI 等外设的控制寄存器。                 |
| `0xE000E000` | 系统控制空间（SCS）         | 包括 NVIC、SysTick 和调试相关寄存器。                   |
### 寄存器
#### R13（栈指针）
   - 可以通过PUSH和POP指令实现栈的访问。
   - 物理上有主栈指针（MSP）和进程栈指针（PSP）两个栈指针。
   - `MSP`是默认的栈指针，用于复位后或处理器处于处理模式时。
   - `PSP`仅用于线程模式。
   - MSP和PSP都是32位的，栈地址必须是32位对齐的。
#### R14（链接寄存器，LR）
   - 用于函数或子程序调用时返回地址的保存。
   - 在异常处理期间，有特殊用途。
#### R15（程序计数器，PC） 
> PC的最低位必须置`1`，以表示`Thumb`状态，否则会引发异常。

   - 是可读写的寄存器。
   - 读操作可以获取当前指令地址加`4`。
   - 写操作会引起指令跳转。
### 向量表
> 向量地址的最低位必须为`1`，表明是`Thumb`状态。

- 向量表在地址空间中的位置是可以设置的，通过 NVIC 中的**重定位寄存器**来指出向量表的地址。
- 在复位后，重定位寄存器的值为`0`，所以，在地址`0`（即 `FLASH` 地址0）处必须包含一张向量表，用于复位后的异常分配。

## STM32 F103 启动流程
#todo
### **1. 电源上电与复位**

- **芯片上电后：**
    
    - 电压逐步稳定。
    - 复位电路释放复位信号。
    - ARM 核心开始从固定的复位向量地址读取指令。
- **复位后的硬件状态：**
    
    - 寄存器初始化为默认值。
    - 指令从固定的复位向量地址开始执行。
    - 大部分外设处于未初始化状态。
### **2. 启动向量表**

在 ARM Cortex-M 系列中，**启动向量表（Vector Table）** 通常位于 Flash 的起始地址（`0x08000000`）。

- 启动向量表的前两项：
    1. **栈指针初始值（MSP，Main Stack Pointer）：**
        - 位于启动向量表的第一个地址（`0x08000000`）。
        - ARM 启动时会自动将此值加载到 `MSP`。
    2. **复位向量地址（Reset Vector）：**
        - 位于启动向量表的第二个地址（`0x08000004`）。
        - 指向复位后要执行的第一条指令的地址。

### **3. 复位处理（Reset Handler）**

复位向量指向的地址通常是复位处理程序（Reset Handler）。  
复位处理程序完成以下步骤：

1. **初始化数据段：**
    - 将 `.data` 段从 Flash 中复制到 RAM 中。
    - 将 `.bss` 段（未初始化全局变量）清零。
2. **设置系统时钟：**
    - 配置晶振和 PLL，使系统进入目标工作频率。
3. **初始化外设：**
    - 启动基本外设（如 GPIO、时钟模块等）。
4. **跳转到主程序：**
    - 调用 `main()` 函数开始执行应用代码。
### **4. 执行主程序**

完成复位处理后，执行用户代码的主函数 `main()`，程序进入正常运行状态。
### 5. 中断与异常处理
在 ARM Cortex-M 系列中：

1. 中断向量表指向各个中断的服务函数。
2. 中断或异常发生时，处理器通过 NVIC 硬件控制器进行中断管理。
3. 系统中断执行完成后，返回到中断前的执行点。
## 启动文件 start.s
启动文件由汇编编写，是系统上电复位后第一个执行的程序。
### 启动文件作用
1. 初始化堆栈指针`SP`（`__initial_sp`）
2. 初始化`PC`指针（`Reset_Handler`）
3. 初始化中断向量表（`__Vectors`）
4. 配置系统时钟（`SystemInit`）
5. 调用`C`库函数`_main`初始化用户堆栈，从而最终调用`main`函数。
### 启动文件使用的ARM汇编指令汇总

| 指令名称          | 作用                                                                                    |
| ------------- | ------------------------------------------------------------------------------------- |
| EQU           | 给数字常量取一个符号名，相当于C语言中的define                                                            |
| AREA          | 汇编一个新的代码段或者数据段                                                                        |
| SPACE         | 分配内存空间                                                                                |
| PRESERVE8     | 当前文件堆栈需按照8字节对齐                                                                        |
| EXPORT        | 声明一个标号具有全局属性，可被外部的文件使用                                                                |
| DCD           | 以字为单位分配内存，要求4字节对齐，并要求初始化这些内存                                                          |
| PROC          | 定义子程序，与ENDP成对使用，表示子程序结束                                                               |
| WEAK          | 弱定义，如果外部文件声明了一个标号，则优先使用外部文件定义的标号， 如果外部文件没有定义也不出错。要注意的是：这个不是ARM的指令，是编译器的，这里放在一起只是为了方便。 |
| IMPORT        | 声明标号来自外部文件，跟C语言中的EXTERN关键字类似                                                          |
| B             | 跳转到一个标号                                                                               |
| ALIGN         | 编译器对指令或者数据的存放地址进行对齐，一般需要跟一个立即数，缺省表示4字节对齐 要注意的是：这个不是ARM的指令，是编译器的，这里放在一起只是为了方便。         |
| END           | 到达文件的末尾，文件结束                                                                          |
| IF,ELSE,ENDIF | 汇编条件分支语句，跟C语言的if else类似                                                               |
### Stack
```asm
Stack_Size      EQU     0x00000400  ;指定栈大小为1KB

                AREA    STACK, NOINIT, READWRITE, ALIGN=3  ;名字为STACK，NOINIT即不初始化，可读可写，8（2^3）字节对齐。
Stack_Mem       SPACE   Stack_Size  ;分配Stack_Size大小的空间
__initial_sp                        ;紧挨着SPACE语句放置，表示栈的结束地址，即栈顶地址，栈是由高向低生长的。
```
### Heap
```asm
Heap_Size       EQU     0x00000200  ;指定堆大小为512Bytes

                AREA    HEAP, NOINIT, READWRITE, ALIGN=3  ;名字为HEAP，NOINIT即不初始化，可读可写，8（2^3）字节对齐。
__heap_base                         ;堆的起始地址
Heap_Mem        SPACE   Heap_Size   ;申请内存空间
__heap_limit                        ;堆的结束地址
```

```asm
PRESERVE8                           ;指定当前文件的堆栈按照8字节对齐。
THUMB                               ;表示后面指令兼容THUMB指令。
```

THUBM是ARM以前的指令集，16bit， 现在Cortex-M系列的都使用THUMB-2指令集，THUMB-2是32位的，兼容16位和32位的指令，是THUMB的超集。
### 向量表
```asm
AREA    RESET, DATA, READONLY       ;定义一个数据段，名字为RESET，可读。
EXPORT  __Vectors                   ;声明 __Vectors标号具有全局属性
EXPORT  __Vectors_End
EXPORT  __Vectors_Size
```

**EXPORT**： 声明一个标号可被外部的文件使用，使标号具有全局属性。如果是IAR编译器，则使用的是GLOBAL这个指令。

当内核响应了一个发生的异常后，对应的异常服务例程(`ESR`)就会执行。为了决定 ESR 的入口地址， 内核使用了“向量表查表机制”。这里使用一张向量表。向量表其实是一个 `WORD`（ 32 位整数）数组，每个下标对应一种异常，该下标元素的值则是该 ESR 的入口地址。向量表在地址空间中的位置是可以设置的，通过 NVIC 中的一个重定位寄存器来指出向量表的地址。在复位后，该寄存器的值为 0。因此，在地址 0 （即FLASH 地址0）处必须包含一张向量表，用于初始时的异常分配。要注意的是这里有个另类： 0 号类型并不是什么入口地址，而是给出了复位后 `MSP` 的初值。

**DCD**：分配一个或者多个以字为单位的内存，以**四字节**对齐，并要求初始化这些内存。在向量表中，DCD分配了一堆内存，并且以ESR的入口地址初始化它们。
```asm
 __Vectors  DCD   __initial_sp                ;栈顶地址
		 DCD   Reset_Handler                  ;复位程序地址
         DCD   NMI_Handler
         DCD   HardFault_Handler
         DCD   MemManage_Handler
         DCD   BusFault_Handler
         DCD   UsageFault_Handler
         DCD   0                              ;0 表示保留
         DCD   0
         DCD   0
         DCD   0
         DCD   SVC_Handler
         DCD   DebugMon_Handler
         DCD   0
         DCD   PendSV_Handler
         DCD   SysTick_Handler

		 ;外部中断开始
         DCD   WWDG_IRQHandler
         DCD   PVD_IRQHandler
         DCD   TAMPER_IRQHandler

		 ;省略
         DCD   DMA2_Channel2_IRQHandler
         DCD   DMA2_Channel3_IRQHandler
         DCD   DMA2_Channel4_5_IRQHandler
 __Vectors_End                                ;向量表结束地址
 __Vectors_Size EQU __Vectors_End - __Vectors
```
### 复位程序
```asm
AREA    |.text|, CODE, READONLY  ;定义一个名称为.text的代码段，只读。
```

**WEAK**：表示弱定义，如果外部文件优先定义了该标号则首先引用该标号，如果外部文件没有声明也不会出错。这里表示复位子程序可以由用户在其他文件重新实现，这里并不是唯一的。

**IMPORT**：表示该标号来自外部文件，跟C语言中的EXTERN关键字类似。这里表示SystemInit和__main这两个函数均来自外部的文件。

`SystemInit()`是一个标准的库函数，在system_stm32f103xe.c这个库文件中定义。主要作用是配置系统时钟，这里调用这个函数之后，单片机的系统时钟配被配置为`72M`。

`__main`是一个标准的C库函数，主要作用是[[STM32 启动流程#用户堆栈初始化|初始化用户堆栈]]，并在函数的最后调用main函数去到C的世界。这就是为什么我们写的程序都有一个main函数的原因。
```asm
Reset_Handler PROC
            EXPORT  Reset_Handler    [WEAK]
            IMPORT  SystemInit
            IMPORT  __main

            LDR     R0, =SystemInit          ;初始化系统时钟
            BLX     R0
            LDR     R0, =__main              ;调用c库函数__main初始化用户堆栈
            BX      R0
            ENDP
```

> CM4内核指令

| 指令  | 作用                                                 |
| --- | -------------------------------------------------- |
| LDR | 从存储器中加载字到一个寄存器中                                    |
| BL  | 跳转到由寄存器/标号给出的地址，并把跳转前的下条指令地址保存到LR                  |
| BLX | 跳转到由寄存器给出的地址，并根据寄存器的LSE确定处理器的状态，还要把跳转前的下条指令地址保存到LR |
| BX  | 跳转到由寄存器/标号给出的地址，不用返回                               |
| B   | 跳转到一个标号                                            |
### 中断服务程序
在启动文件里面已经帮我们写好所有中断的中断服务函数，真正的中断复服务程序需要我们在外部的`C`文件里面重新实现，这里只是提前占了一个位置。
```asm
NMI_Handler     PROC                                      ;系统异常
                EXPORT  NMI_Handler           [WEAK]
                B       .                                 ;跳转到`.`，表示无限循环
                ENDP

;限于篇幅，中间代码省略
SysTick_Handler PROC
                EXPORT  SysTick_Handler       [WEAK]
                B       .
                ENDP

Default_Handler PROC    ;外部中断
                EXPORT  WWDG_IRQHandler       [WEAK]
                EXPORT  PVD_IRQHandler        [WEAK]
                EXPORT  TAMP_STAMP_IRQHandler [WEAK]

;限于篇幅，中间代码省略
LTDC_IRQHandler
LTDC_ER_IRQHandler
DMA2D_IRQHandler
                B       .
                ENDP
```
### 用户堆栈初始化
```asm
	;用户栈和堆初始化,由C库函数_main来完成
    IF      :DEF:__MICROLIB               ;这个宏在KEIL里面开启

    EXPORT  __initial_sp
    EXPORT  __heap_base
    EXPORT  __heap_limit

    ELSE

    IMPORT  __use_two_region_memory       ;这个函数由用户自己实现
    EXPORT  __user_initial_stackheap

__user_initial_stackheap

    LDR     R0, =  Heap_Mem
    LDR     R1, =(Stack_Mem + Stack_Size)
    LDR     R2, = (Heap_Mem +  Heap_Size)
    LDR     R3, = Stack_Mem
    BX      LR

    ALIGN                                 ;地址对齐，后面会跟一个立即数。缺省表示4字节对齐。

    ENDIF
    END                                   ;文件结束。
```
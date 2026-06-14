 - [ESP32-C6 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-c6_datasheet_cn.pdf) (PDF)--资料来源
- ESP32-C6 有三个 UART 接口，即 UART0，UART1 和 LP UART。三个 UART 均支持 CTS 和 RTS 信号的硬件流 控以及软件流控（XON 和 XOFF）。
- UART0 和 UART1 支持异步通信（RS232 和 RS485）和 IrDA，通信速率可达到 5 Mbps。
- UART0 和 UART1 接口 通过共用的 UHCI0 接口（即通用主机控制接口）与 GDMA 相连，可被 GDMA 访问或者 CPU 直接访问。
- LP UART 仅支持异步通信（RS232），通信速率可达到 1.25 Mbps。LP UART 只支持 CPU 直接访问。
- UART 还可以用于红外数据交换 (IrDA) 或 RS485 调制解调器。
- ESP32-C6 中的两个 UART 接口通过通用主机控制器接口（UHCI）共用 1 组 GDMA TX/RX 通道。在 GDMA 模 式下，支持对 HCI 协议数据包的解析（decoder）及数据包封装（encoder）。

### LP_UART
- LP IO MUX 功能在 HP 数字系统关闭时激活，从而节省功耗。LP IO MUX 的功能及数据的输入/输出由 LP CPU 配置。

![[../../图片文档/LP_UART 引脚图.png]]
### UART0
- I – 输入。O – 输出。T – 高阻。
- I1 – 输入；如果该管脚分配了 Fn 以外的功能，则 Fn 的输入信号恒为 1。
- I0 – 输入；如果该管脚分配了 Fn 以外的功能，则 Fn 的输入信号恒为 0。
![[../../图片文档/UART0引脚图.png]]
#### 修改uart0的默认引脚
##### ESP32的调试输出被中断，这就是为什么看不到进入main_task的日志信息。
- UART0默认被用于编程和打印调试信息。如果你在代码中重新配置了UART0，可能会影响到这些功能，包括打印到串行监视器的日志信息。
###### uart0
```c
I (262) sleep: Configure to isolate all GPIO pins in sleep state
I (268) sleep: Enable automatic switching of GPIO sleep configuration
I (275) coexist: coex firmware version: 80b0d89
I (281) coexist: coexist rom version 5b8dcfa
```
###### uart1
```c
I (262) sleep: Configure to isolate all GPIO pins in sleep state
I (268) sleep: Enable automatic switching of GPIO sleep configuration
I (275) coexist: coex firmware version: 80b0d89
I (281) coexist: coexist rom version 5b8dcfa
I (286) app_start: Starting scheduler on CPU0
I (291) main_task: Started on CPU0
I (291) main_task: Calling app_main()
```

### UART1
- UART0 和 UART1 通过共用的 UHCI0 接口与 GDMA 相连，可以被 GDMA 访问或者 CPU 直接访问。这意味着 UART0 和 UART1 可以同时使用。
- 需要确保代码能够正确地管理和调度这两个 UART 接口，以避免冲突和数据丢失。同时，还需要确保操作系统或者驱动程序支持同时使用多个 UART 接口。
- 需要考虑冲突问题[[看门狗超时]]
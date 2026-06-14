`Linux`内核提供了`pinctrl`子系统和`gpio`子系统用于`GPIO`驱动。`pinctrl`子系统就是设置引脚的复用关系和电气属性的，`GPIO`子系统就是当`pinctrl`子系统将复用关系设置为`GPIO`以后，我们使用`GPIO`子系统来操作我们的`GPIO`

`GPIO`子系统在之前的内核中也是存在的，但是`pinctrl`子系统的加入`GPIO`子系统也是有很大的改变，之前的`GPIO`子系统需要芯片厂商提供的`mach`文件，而加入设备树后，`GPIO`子系统使用设备树来实现。
## Pinctrl子系统
> `pinctrl`子系统是随着设备树的加入而加入的，依赖于设备树。
- 源码目录`drivers/pinctrl`
- `NXP`的`pinctrl`子系统配置如下（不同厂家的`pin controller`的节点里面的属性都可以通过`Linux`源码`/Documentation/devicetree/bindings`下的`txt`文档查看。）
```c
pinctrl_tsc: tscgrp {
    fsl,pins = <
        MX6UL_PAD_GPIOIO1_IO01__GPIO1_IO01 0xb0
        MX6UL_PAD_GPIOI_O1_IO02__GPIO1_IO02 0xb0
        MX6UL_PAD_GPIO1_I_O03__GPIO1_IO03 0xb0
        MX6UL_PAD_GPIO1_I_O04__GPIO1_IO04 0xb0
    >;
};

pinctrl_uart1: uartgrp {
    fsl,pins = <
        MX6UL_PAD_UART1_TX_DATA__UART1_DCE_TX 0x1b0b1
        MX6UL_PAD_UART1_RX_DATA__UART1_DCE_RX 0x1b0b1
    >;
};

pinctrl_uart2: uart2grp {
    fsl,pins = <
        MX6UL_PAD_UART2_TX_DATA__UART2_DCE_TX 0x1b0b1
        MX6UL_PAD_UART2_RX_DATA__UART2_DCE_RX 0x1b0b1
        MX6UL_PAD_UART3_RX_DATA__UART2_DCE_RTS 0x1b0b1
        MX6UL_PAD_UART3_TX_DATA__UART2_DCE_CTS 0x1b0b1
    >;
};
```

### 主要作用
- *获取设备树中pin信息*：管理系统中所有可控制的`pin`，在系统初始化时，枚举所有可以控制的`pin`，并标识。
- *设置pin的引脚复用功能*：若个引脚可以组成`pin group`，实现特定功能
- *设置pin电气特性*：比如上拉、速度、驱动能力
> 只需在设备树中设置对应`pin`的属性，其余初始化工作由`pinctrl`子系统完成
### 配置
以设备树文件中的`iomux`节点为例
```shell
panel@0 {
                compatible = "itop_mipi_screen";
                reg = <0>;
                pinctrl-0 = <&pinctrl_mipi_dsi_en>;
                reset-gpio = <&gpio1 8 GPIO_ACTIVE_HIGH>;
                dsi-lanes = <4>;
                video-mode = <2>;       /* 0: burst mode
                                        * 1: non-burst mode with sync event
                                        * 2: non-burst mode with sync pulse
                                        */
                panel-width-mm = <68>;
                panel-height-mm = <121>;
 
                status = "okay";/*"okay";*/
 
&iomuxc
{
    pinctrl_mipi_dsi_en: mipi_dsi_en {
            fsl,pins = <
                MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8       0x16
            >;
        };
        ......
    }；
```
`iomuxc` 节点就是 `I.MX8MM`的 `IOMUXC` 外设对应的节点，不同的外设使用的`PIN`不同，配置也不同

- `GPIO1_IO08`的配置信息如下所示：
```shell
MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8 0x16
```
- 引脚的配置部分分为俩部分：`MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8`和`0x16`
- 对于一个引脚来说，配置主要包括两方面
- 一个是设置这个引脚的*复用*功能，一个是设置这个引脚的*电气*特性
- `MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8`和`0x16`，一个是设置`GPIO1_IO08`的复用功能，一个是用来设置`GPIO1_IO08`的电气特性。
- `MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8`，这是一个*宏定义*，定义在`linux/uboot-imx/arch/arm/dts/include/dt-bindings/pinctrl/pins-imx8mm.h`文件中。设备树文件会引用这个头文件。
- `MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8`的宏定义内容如下所示
```c
#define MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO08              0x048 0x2B0 0x0000 0x00 0x00 
#define MX8MM_IOMUXC_GPIO1_IO08_ENET1_1588_EVENT0_IN    0x048 0x2B0 0x0000 0x01 0x00 
#define MX8MM_IOMUXC_GPIO1_IO08_CCM_SRC_GP_CMIX_WAIT    0x048 0x2B0 0x0000 0x05 0x00 
#define MX8MM_IOMUXC_GPIO1_IO08_QSPI_TEST_TRIG          0x048 0x2B0 0x0000 0x06 0x00
```
通过参考手册可知：上述宏定义分别对应`GPIO1_IO08`这个引脚的`8`个复用`IO`
- `MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8`表示将`GPIO1_IO8`这个`IO`复用为`GPIO1_IO8`
- *0x048*：`mux_reg` *复用寄存器*的偏移地址，设备树中的 `iomuxc` 节点就是 `IOMUXC` 外设对应的节点，根据其 `reg` 属性可知 `IOMUXC` 外设寄存器起始地址为 `0x30330000`。-->`MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8` 复用寄存器地址为`0x30330000+0x0048=0x30330048`
- *0x02B0*：`conf_reg` 寄存器偏移地址，同`mux_reg`，寄存器`IOMUXC_SW_PAD_CTL_PAD_GPIO1_IO08`的地址`0x30330000+0x02B0=0x303302B0`。
- *0x000*：`input_reg`寄存器偏移地址，有些外设有 `input_reg` 寄存器，没有的话就不需要设置。
- *0x0*：`mux_reg` 寄存器值，用于设置 `IO` 复用*模式*，这里设置为 `0x0`，就相当于设置复用功能为`ENET1_RX_EN`。
- *0x0*：`input_reg` 寄存器值，在这里无效。
---
- `MX8MM_IOMUXC_GPIO1_IO08_GPIO1_IO8` *0x16*： `config_reg`寄存器是设置一个`PIN `的*电气特性*的，由用户设置，实现设置一个`IO`的上下拉，驱动能力，和速度。
### 调用
> 调用 `pinctrl` 一般在设备树中进行
#### 配置引脚为 GPIO的通用流程
- 在 `IOMUXC/pinctrl` 中对某一个引脚进行配置，两个部分（**复用**、**初始化**），在外设节点中调用
- 在驱动中获取*设备节点*以及 `GPIO`
- 对`GPIO`进行配置
#### 使用`pin controller`里面定义的节点
```c
pinctrl-names = "default"; 
pinctrl-0 = <&pinctrl_hog_1>;
```
- `pinctrl-names = "default";` 设备的*状态*，可以有多个状态，`default`为状态`0`
- `pinctrl-0 = <&pinctrl_hog_1>;`第`0`个状态所对应的引脚配置，也就是`default`状态对应的引脚在`pin controller`里面定义好的节点`pinctrl_hog_1`里面的管脚配置。
---
```c
pinctrl-names = "default"，"wake up";
pinctrl-0 = <&pinctrl_hog_1>;
pinctrl-1 = <&pinctrl_hog_2>;
```
- `pinctrl-names = "default"，"wake up"; ` default为状态0，`wake up`为状态`1`
- `pinctrl-1 = <&pinctrl_hog_2>;`第`1`个状态所对应的引脚配置，也就是`wake up`状态对应的引脚在`pin controller`里面定义好的节点`pinctrl_hog_2`里的管脚配置
---
```c
pinctrl-names = "default";
pinctrl-0 = <&pinctrl_hog_1
&pinctrl_hog_2>;
```
- `pinctrl-names = "default";` 设备的状态，可以有多个状态，`default`为状态`0`
- `pinctrl-0 = <&pinctrl_hog_1  &pinctrl_hog_2>;`第`0`个状态所对应的引脚配置，也就是`default`状态对应的引脚在`pin controller`里面定义好的节点,对应`pinctrl_hog_1`和`pinctrl_hog_2`这俩个节点的管脚配置。
## GPIO子系统
> 当我们使用`pinctrl`子系统将引脚的复用关系设置为`GPIO`以后，就可以使用`GPIO`子系统来操作`GPIO`
#### 作用
> 在引入*设备树*后对`GPIO`子系统进行了大的改造，使用设备树来实现并提供*统一的接口*。通过 `GPIO` 子系统功能要实现：

- 引脚功能的*配置*：设置为 `GPIO`，特殊功能，`GPIO` 的方向，设置为中断等
- 实现*软硬件的分离*：分离出硬件差异，有厂商提供的底层支持；软件分层。驱动只需要调用接口 `API` 即可操作 `GPIO`
- `iommu` *内存管理*：直接调用宏即可操作 `GPIO`

通过 `GPIO` 子系统，我们可以通过如下的方式来配置 `GPIO`
```c
gpio_request(leddev.led0, "led0");
gpio_direction_output(leddev.led0, 1);
gpio_set_value(leddev.led0, 0);
gpio_direction_input(key.irqkeydesc[0].gpio);
gpio_get_value(keydesc->gpio);
gpio_free(leddev.led0);
```
### 常用函数
#### gpio_request()
```c
/**
 * gpio_request - Request an GPIO for use by the caller
 * @gpio: The GPIO number to request
 * @label: A label for the consumer of this GPIO
 *
 * This function is used to request an GPIO for use by the caller. The
 * GPIO number can be obtained from the device tree using the
 * of_get_named_gpio function, which returns the GPIO number.
 * The label is used to identify the consumer of this GPIO.
 * Returns 0 if the request is successful, or another value if it fails.
 */
int gpio_request(unsigned gpio, const char *label);
```
#### gpio_free()
```c
/**
 * gpio_free - Release an GPIO previously requested by gpio_request
 * @gpio: The GPIO number to release
 *
 * This function is used to release an GPIO that is no longer needed.
 * It should be called when the GPIO is no longer required by the caller.
 */
void gpio_free(unsigned gpio);
```
#### gpio_direction_input()
```c
/**
 * gpio_direction_input - Set the direction of an GPIO to input
 * @gpio: The GPIO number to set as input
 *
 * This function is used to set the direction of an GPIO to input.
 * Returns 0 if the direction is set successfully, or a negative value if it fails.
 */
int gpio_direction_input(unsigned gpio);
```
#### gpio_direction_output()
```c
/**
 * gpio_direction_output - Set the direction of an GPIO to output and set the default output value
 * @gpio: The GPIO number to set as output
 * @value: The default output value for the GPIO
 *
 * This function is used to set the direction of an GPIO to output and
 * set the default output value. Returns 0 if the direction and value
 * are set successfully, or a negative value if it fails.
 */
int gpio_direction_output(unsigned gpio, int value);
```
#### gpio_get_value()
```c
/**
 * gpio_get_value - Get the value of an GPIO
 * @gpio: The GPIO number to get the value from
 *
 * This function is used to get the value of an GPIO. It returns the
 * GPIO value (0 or 1) if successful, or a negative value if it fails.
 */
int __gpio_get_value(unsigned gpio);
```
#### gpio_set_value()
```c
/**
 * gpio_set_value - Set the value of an GPIO
 * @gpio: The GPIO number to set the value for
 * @value: The value to set the GPIO to
 *
 * This function is used to set the value of an GPIO. It does not return
 * a value, but will perform the operation on the specified GPIO.
 */
void __gpio_set_value(unsigned gpio, int value);
```
#### of_get_named_gpio()
```c
/**
 * of_get_named_gpio - Get the GPIO number from the device tree
 * @np: The device node to get the GPIO number from
 * @propname: The property name containing the GPIO information
 * @index: The index of the GPIO to get the number for
 *
 * This function is used to get the GPIO number from the device tree.
 * It returns the GPIO number if successful, or a negative value if it fails.
 * The GPIO number is required for other GPIO API functions in the Linux kernel.
 */
int of_get_named_gpio(struct device_node *np, const char *propname, int index);
```

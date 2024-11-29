## 定义
> `I2C`（`Inter-Integrated Circuit`）是一种串行通信总线协议，用于在集成电路（IC）之间进行通信。
- 由两根线组成：串行数据线（`SDA`）和串行时钟线（`SCL`）。
- `I2C`总线采用*主从*结构，可以支持*多个设备*在同一条总线上进行通信。

常用于读取硬件监视器、传感器、[实时时钟](https://en.wikipedia.org/wiki/Real-time_clock "Real-time clock")、控制执行器、访问低速[DAC](https://en.wikipedia.org/wiki/Digital-to-analog_converter "Digital-to-analog converter")和[ADC](https://en.wikipedia.org/wiki/Analog-to-digital_converter "Analog-to-digital converter")、控制简单的[LCD](https://en.wikipedia.org/wiki/Liquid_crystal_display "Liquid crystal display")或[OLED](https://en.wikipedia.org/wiki/Organic_light-emitting_diode "Organic light-emitting diode")显示器、通过[显示数据通道更改](https://en.wikipedia.org/wiki/Display_Data_Channel "Display Data Channel")[计算机显示](https://en.wikipedia.org/wiki/Computer_display "Computer display")设置（例如背光、对比度、色调、色彩平衡）以及更改扬声器音量。
### 总线结构
> 由``SDA``(串行数据线)和`SCL`(串行时钟线)及上拉电阻组成。

在总线==空闲==状态时，这两根线一般上拉电阻拉高，保持着**高电平**。
### `I2C`传输中有几种信号
> 三种信号：开始信号、应答信号、结束信号
### `I2C`传输一个`byte`需要几个`CLK`
- 9个
### I2C从机地址
I²C的参考设计使用一个`7`比特长度的地址空间但保留了`16`个地址，所以在一组总线最多可和`112`个节点通信。
从机地址也可选择`10`比特长度。
#### 两个设备地址冲突
- *单独的I2C总线*：需要额外硬件资源。
- *修改设备地址*：某些`I²C`设备支持通过硬件跳线（`ADDR`引脚）或命令改变地址。
### I2C速率
| Mode                  | Maximum  <br>speed | Maximum  <br>capacitance                              | Drive                                                                                  | Direction      |
| --------------------- | ------------------ | ----------------------------------------------------- | -------------------------------------------------------------------------------------- | -------------- |
| Standard mode (Sm)    | 100 kbit/s         | 400 [pF](https://en.wikipedia.org/wiki/Farad "Farad") | [Open drain](https://en.wikipedia.org/wiki/Open_collector "Open collector")*           | Bidirectional  |
| Fast mode (Fm)        | 400 kbit/s         | 400 pF                                                | Open drain*                                                                            | Bidirectional  |
| Fast mode plus (Fm+)  | 1 Mbit/s           | 550 pF                                                | Open drain*                                                                            | Bidirectional  |
| High-speed mode (Hs)  | 1.7 Mbit/s         | 400 pF                                                | Open drain*                                                                            | Bidirectional  |
| High-speed mode (Hs)  | 3.4 Mbit/s         | 100 pF                                                | Open drain*                                                                            | Bidirectional  |
| Ultra-fast mode (UFm) | 5 Mbit/s           | ?                                                     | [Push–pull](https://en.wikipedia.org/wiki/Push%E2%80%93pull_output "Push–pull output") | Unidirectional |
### I2C数据位
每次发送`8`位，一般`MSB`(最高位）最先通过SDA传输，每发送一个字节，对应设备都需要回复一个`ack`确定位。
- `1`：当SCL为**高电平**时，SDA保持为高电平。
- `0`：当SCL为高电平时，SDA保持为低电平。
### I2C确认位
- **ACK**
    - 发送方发出地址或数据，接收方拉低 SDA 线确认收到。
    - 通信继续进行。

- **NACK**
	- 从设备无法识别主设备的地址。
	- 接收数据缓冲区已满。
	- 数据传输完成后，从设备用 NACK 表示不再需要数据。
## `I2C`数据的流程
数据通过单条 `SDA` 数据线，通过 `0` 和 `1`（位）的图案化序列在主设备和从设备之间传输。

每个 `0` 和 `1` 的序列称为一个**事务**，每个事务中的数据结构如下：
- *开始信号*：`SCL` 为高电平时，`SDA` 由高电平向低电平跳变 `-->`开始传输数据
- *设备地址*：`7`位==从机地址==`-->`选择设备
- 再给地址添加一个**方向位**用来表示接下来数据传输的方向（`0`表述主-->从`wirte`）（`1`表示从-->主`read`）
- *寄存器地址*：等价于数据
- *数据*：由 `8` 位组成，它们由发送方设置，以及需要传输到接收方的数据位。后跟一个 `ACK​​/NACK` 位。
- *ACK/NACK位*：代表已确认/未确认位（**应答信号**）。如果任何从设备的物理地址与主设备广播的地址一致，则该位的值被从设备设置为“`0`”。否则，它保持在逻辑“`1`”（默认）。
- *结束信号*：`SCL` 为高电平时，`SDA` 由低电平向高电平跳变 `-->`停止传输数据。
## 时序

### 写时序
![[I2C写数据帧-以24C02为例.png]]
### 读时序
数据部分从机**主动**控制总线发送数据给主机，然后主机来进行应答，刚好与`IIC`写数据相反。

在通信过程中需要进行读写切换时不需要发送停止，而是应答以后重新发一次**起始**和从机地址及读写状态，接着进行下面的数据处理即可。
![[I2C读数据帧-以24C02为例.png]]
## 硬件电路
### 开漏输出->多设备共用总线不烧毁电路
- 两条 `I2C` 总线（`SDA`、`SCL`）都作为开漏驱动器运行。
- `I2C` 网络上的任何设备/IC 都可以将 `SDA` 和 `SCL` 驱动为低电平，但不能将它们驱动为*高电平*。
- 使用开漏系统的原因是允许多个节点连接到总线，而不会因信号争用而发生**短路**。
### 上拉电阻->输出高电平
每条线都需要一个[上拉电阻](https://en.wikipedia.org/wiki/Pull-up_resistor "上拉电阻")。通过将线拉至地可输出逻辑“0”，通过让线浮动（输出[高阻抗](https://en.wikipedia.org/wiki/High_impedance "高阻抗")）以便上拉电阻将其拉高可输出逻辑“1”。线永远不会**主动驱动**为高电平。

**空闲电平**：SCL 和 SDA 均为高电平。
### 线与
多个节点可能同时驱动线路。如果_任何_节点将线路驱动为低电平，则该节点将为低电平。尝试传输逻辑 1（即让线路浮动为高电平）的节点可以检测到这种情况，并断定另一个节点同时处于活动状态。
#### SDA-仲裁
在SDA上使用时，线与被称为**仲裁**，确保每次只有一个发送器。

每个控制器都会监视总线上的起始位和停止位，并且当另一个控制器使总线保持繁忙时，不会启动消息。但是，两个控制器可能会大约同时开始传输；在这种情况下，会发生仲裁。当一个控制器寻址多个目标时，目标传输模式也可以进行仲裁，但这种情况不太常见。I2C具有确定性的仲裁策略。每个发送器都会检查数据线（SDA）的电平并将其与预期电平进行比较；如果它们不匹配，则该发送器已**失去仲裁**并退出此协议交互。
#### SCL-时钟延展
已寻址的目标设备在接收（或发送）一个字节后可能会将时钟线 (SCL) 保持在**低位**，表示它尚未准备好处理更多数据。

与目标通信的控制器可能未完成当前位的传输，而必须等到时钟线实际变为高位。如果目标正在时钟延长，则时钟线仍将处于低位（因为连接是开漏的）。如果第二个较慢的控制器试图同时驱动时钟，情况也是如此。（如果有多个控制器，通常除了其中一个之外，其他所有控制器都会失去仲裁。）

控制器必须等到观察到*时钟线升高*，并等待最短时间（标准 100 kbit/s  I2C为 4 μs）后才能再次将时钟拉低。
## I2C模拟
用iic的场景，一般是低速通信，不超过400k，和外设通信，数据量不超过几十个字节，通过SCL时钟驱动，ack握手有有等待时间，还没碰到过时序问题，最多时钟周期不是稳定值，已经好几年多个项目验证了，还没出过问题，适配过至少5种外设，通信频率全部是400K以下的，从来不关中断。

部分产品也是全年无休运行，还没出过问题。在单片机，软核对硬核fpga上跑，都没出过问题。

模拟iic还有一个好处是可以灵活的处理ack超时，提前发停止条件终止通信。


```C
// Hardware-specific support functions that MUST be customized:
#define I2CSPEED 100
void I2C_delay(void);
bool read_SCL(void); // Return current level of SCL line, 0 or 1
bool read_SDA(void); // Return current level of SDA line, 0 or 1
void set_SCL(void); // Do not drive SCL (set pin high-impedance)
void clear_SCL(void); // Actively drive SCL signal low
void set_SDA(void); // Do not drive SDA (set pin high-impedance)
void clear_SDA(void); // Actively drive SDA signal low
void arbitration_lost(void);
bool started = false; // global data
void i2c_start_cond(void)
{
  if (started) { 
    // if started, do a restart condition
    // set SDA to 1
    set_SDA();
    I2C_delay();
    set_SCL();
    while (read_SCL() == 0) { // Clock stretching
      // You should add timeout to this loop
    }
    // Repeated start setup time, minimum 4.7us
    I2C_delay();
  }
  if (read_SDA() == 0) {
    arbitration_lost();
  }
  // SCL is high, set SDA from 1 to 0.
  clear_SDA();
  I2C_delay();
  clear_SCL();
  started = true;
}
void i2c_stop_cond(void)
{
  // set SDA to 0
  clear_SDA();
  I2C_delay();
  set_SCL();
  // Clock stretching
  while (read_SCL() == 0) {
    // add timeout to this loop.
  }
  // Stop bit setup time, minimum 4us
  I2C_delay();
  // SCL is high, set SDA from 0 to 1
  set_SDA();
  I2C_delay();
  if (read_SDA() == 0) {
    arbitration_lost();
  }
  started = false;
}
// Write a bit to I2C bus
void i2c_write_bit(bool bit)
{
  if (bit) {
    set_SDA();
  } else {
    clear_SDA();
  }
  // SDA change propagation delay
  I2C_delay();
  // Set SCL high to indicate a new valid SDA value is available
  set_SCL();
  // Wait for SDA value to be read by target, minimum of 4us for standard mode
  I2C_delay();
  while (read_SCL() == 0) { // Clock stretching
    // You should add timeout to this loop
  }
  // SCL is high, now data is valid
  // If SDA is high, check that nobody else is driving SDA
  if (bit && (read_SDA() == 0)) {
    arbitration_lost();
  }
  // Clear the SCL to low in preparation for next change
  clear_SCL();
}
// Read a bit from I2C bus
bool i2c_read_bit(void)
{
  bool bit;
  // Let the target drive data
  set_SDA();
  // Wait for SDA value to be written by target, minimum of 4us for standard mode
  I2C_delay();
  // Set SCL high to indicate a new valid SDA value is available
  set_SCL();
  while (read_SCL() == 0) { // Clock stretching
    // You should add timeout to this loop
  }
  // Wait for SDA value to be written by target, minimum of 4us for standard mode
  I2C_delay();
  // SCL is high, read out bit
  bit = read_SDA();
  // Set SCL low in preparation for next operation
  clear_SCL();
  return bit;
}
// Write a byte to I2C bus. Return 0 if ack by the target.
bool i2c_write_byte(bool send_start,
                    bool send_stop,
                    unsigned char byte)
{
  unsigned bit;
  bool     nack;
  if (send_start) {
    i2c_start_cond();
  }
  for (bit = 0; bit < 8; ++bit) {
    i2c_write_bit((byte & 0x80) != 0);
    byte <<= 1;
  }
  nack = i2c_read_bit();
  if (send_stop) {
    i2c_stop_cond();
  }
  return nack;
}
// Read a byte from I2C bus
unsigned char i2c_read_byte(bool nack, bool send_stop)
{
  unsigned char byte = 0;
  unsigned char bit;
  for (bit = 0; bit < 8; ++bit) {
    byte = (byte << 1) | i2c_read_bit();
  }
  i2c_write_bit(nack);
  if (send_stop) {
    i2c_stop_cond();
  }
  return byte;
}
void I2C_delay(void)
{ 
  volatile int v;
  int i;
  for (i = 0; i < I2CSPEED / 2; ++i) {
    v;
  }
}
```
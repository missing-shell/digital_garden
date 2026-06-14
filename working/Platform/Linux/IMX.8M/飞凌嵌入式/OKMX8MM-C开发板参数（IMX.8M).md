### 概述
- FETMX8MM-C核心板基于[NXP](https://www.forlinx.com/)公司[i.MX8M Min](https://www.forlinx.com/product/imx8m-28.html)i 四核64位处理器设计，
- 主频最高1.8GHz，[Cortex](https://www.forlinx.com/CortexA9.html)-A53架构；
- 2GB DDR4 RAM，支持一个通用型Cortex®-[M4](https://www.forlinx.com/product/imx8m-28.html) 400MHz内核处理器。
- 可提供多种音频接口，包括I2S、AC97、TDM、PDM和SPDIF。
- 提供多种外设接口，如MIPI-CSI、MIPI-DSI、USB、PCIe、UA[RT](https://www.forlinx.com/article-new-c22/276.html)、eC[SPI](https://www.forlinx.com/article-new-c22/245.html)、IIC和千兆以太网。
---
- 具备1080p 60Hz的H.265和VP9解码器； 相比传统的H.264编码，平均解码效率提升50%； 传输和存储同样分辨率的视频所占用带宽和容量是H.264的50%。
- FETMX8MM-C支持Android 9.0和Linux4.14（QT 5.10）两种64位操作系统
- 支持IIS、AC97、TDM、PDM、SPDIF； 支持5个SAI通道，可配置为IIS、AC97、TDM。 支持7.1声道声音输出，及环麦输入，用于高保真音乐系统及语音识别应用。
- FETMX8MM-C具备1个Cortex-M4内核，主频400MHz； 与Cortex-A53通过内部AHB总线通信，可用于低功耗休眠、实时任务处理应用。
#### 目标应用
- IP摄像头视频监控
- 双向视频会议
- 可视门铃
- 图像分析
- 音频处理
- 音频广播系统。
### 参数

|            |                              |                                                                                                          |           |
| ---------- | ---------------------------- | -------------------------------------------------------------------------------------------------------- | --------- |
| 功能         | 数量                           | 参数                                                                                                       |           |
| MIPI_DSI   | 1                            | 4 data lanes；标准规范V1.01r11；最高分辨率范围可达WQHD（1920x1080p60,24bpp）                                              |           |
| MIPI_CSI   | 1                            | 4 lanes；OV5640MIPI                                                                                       |           |
| Audio      | 1                            | `1*MIC`，`1*Phone`，`2*Speaker`                                                                            |           |
| PDM        | 1                            | 由2mm间距10Pin插座引出                                                                                          |           |
| SAI        | 1                            | 由2mm间距26Pin插座引出                                                                                          |           |
| USB Host   | 2                            | 由集线器扩展,USB 2.0（最高支持 480 Mbps）                                                                            |           |
| USB OTG    | 1                            | 标准micro USB插座,USB 2.0 OTG （最高支持 480 Mbps）                                                                |           |
| `Ethernet` | 1                            | 10/100/1000Mbps自适应，RJ-45接口                                                                               |           |
| Mini PCIE  | 1                            | 可外接mini PCIE接口的4G模块。且具有PCIE2.0单通道                                                                        |           |
| WiFi       | 1                            | 模块型号：F23BUUM13-W2  <br><br>WiFi 标准： `IEEE 802.11b/g/n` ，2.4GHz<br><br>Bluetooth：V2.1+EDR/BT v3.0/BT v4.0 |           |
| Bluetooth  | 1                            |                                                                                                          |           |
| TF Card    | 1                            | 兼容SD、SDHC和SDXC（UHS-I）                                                                                    |           |
| SDIO       | 1                            | 由2mm间距20Pin接口插座引出， 8位数据                                                                                  |           |
| SPI        | 2                            | 由2mm间距10Pin插座引出                                                                                          |           |
| QSPI       | 1                            | 板载 16MB QSPI NOR FLASH                                                                                   |           |
| PWM        | 1                            | 接到液晶背光调节                                                                                                 |           |
| IIC        | 4                            | 标准模式100kbits/s的数据速率；快速模式320kbits/s的数据速率。                                                                 |           |
| UART       | 1                            | UART3三线串口，3.3V电平，最高支持 4.0 Mbps；                                                                          |           |
| UART Debug | 1                            | UART2，RS232电平，DB9公头接口，默认波特率115200                                                                        | A53 Debug |
| 1          | UART4，3.3V电平，由2mm间距10Pin插座引出 | M4 Debug                                                                                                 |           |
| RS485      | 1                            | 隔离RS485，最高传输速率500kbps                                                                                    |           |
| JTAG Debug | 1                            |                                                                                                          |           |

#### 开发板接口图

![[Platform/Linux/IMX.8M/飞凌嵌入式/OKMX8MM-C开发板_外设.jpg]]
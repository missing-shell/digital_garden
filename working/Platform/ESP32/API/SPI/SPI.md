* 设置CS信号线的GPIO编号。LCD 驱动程序将使用此 GPIO 来选择 LCD 芯片。如果SPI总线上只有一个设备（即此LCD），则可以将gpio编号设置为`-1`独占总线。
* 握手线（Handshake line）在SPI通信中并不是必须的。它通常用于硬件流控制，以防止数据溢出。在某些情况下，例如当SPI设备的处理速度较慢，不能及时处理接收到的数据时，握手线可以用来暂停或控制数据的发送。

### 流程
1. **包含必要的头文件**：在你的源文件中，你需要包含必要的头文件，如`driver/spi_master.h`。
2. **配置SPI总线**：使用`spi_bus_config_t`结构体来配置SPI总线。这个结构体包含了一些如SCLK、MISO、MOSI等引脚的配置。
3. **初始化SPI总线**：使用`spi_bus_initialize()`函数来初始化SPI总线。
4. **配置SPI设备**：使用`spi_device_interface_config_t`结构体来配置SPI设备。这个结构体包含了一些如时钟速度、模式、队列大小等设备的配置。
5. **添加SPI设备到总线**：使用`spi_bus_add_device()`函数来将设备添加到SPI总线。
6. **创建一个SPI事务**：使用`spi_transaction_t`结构体来创建一个SPI事务。这个结构体包含了一些如长度、发送缓冲区、接收缓冲区等事务的配置。
7. **发送SPI事务**：使用`spi_device_transmit()`函数来发送SPI事务。
8. **处理SPI事务的结果**：在SPI事务发送后，你可以处理接收缓冲区中的数据。
9. **移除SPI设备**：当你不再需要SPI设备时，使用`spi_bus_remove_device()`函数来移除设备。
10. **释放SPI总线**：当你不再需要SPI总线时，使用`spi_bus_free()`函数来释放总线。


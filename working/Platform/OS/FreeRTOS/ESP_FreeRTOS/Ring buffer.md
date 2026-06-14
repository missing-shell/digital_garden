## 概念
### StreamBuffer 和 MassageBuffer的限制：
- 仅支持单一的发送者和单一的接收者
- 数据通过*复制*的方式进行传递
- 无法为**延迟发送**（即发送获取）预留 buffer 空间

> ESP-IDF 提供了一个单独的环形 buffer 来解决上述问题。
### 定义
- ESP-IDF 环形 buffer 是一个典型的 `FIFO buffer`，支持*任意大小*的数据项。
- 在数据项大小*可变*的情况下，环形 buffer 比 FreeRTOS 队列更**节约内存**，可以替代 FreeRTOS 队列使用。
- 环形 buffer 的容量不是由可以存储的数据项*数量*衡量的，而是由用于存储数据项的**内存量**来衡量的。
- 数据项都通过**引用**的方式从环形 buffer 中检索出来，因此，所有检索出的数据项也 **必须** 通过 [`vRingbufferReturnItem()`](https://docs.espressif.com/projects/esp-idf/zh_CN/release-v5.2/esp32s3/api-reference/system/freertos_additions.html#_CPPv421vRingbufferReturnItem15RingbufHandle_tPv "vRingbufferReturnItem") 或 [`vRingbufferReturnItemFromISR()`](https://docs.espressif.com/projects/esp-idf/zh_CN/release-v5.2/esp32s3/api-reference/system/freertos_additions.html#_CPPv428vRingbufferReturnItemFromISR15RingbufHandle_tPvP10BaseType_t "vRingbufferReturnItemFromISR") 返回到环形 buffer，以便将其从环形 buffer 中完全移除。

### 分类
- **不可分割 buffer**：确保将一个数据项存储在连续的内存中，并且在任何情况下都不会尝试分割数据项。当数据项必须占用连续的内存时，请使用不可分割 buffer。 **仅不可分割 buffer 允许为延迟发送保留缓冲空间。** 更多信息请参考函数 [`xRingbufferSendAcquire()`](https://docs.espressif.com/projects/esp-idf/zh_CN/release-v5.2/esp32s3/api-reference/system/freertos_additions.html#_CPPv422xRingbufferSendAcquire15RingbufHandle_tPPv6size_t10TickType_t "xRingbufferSendAcquire") 和 [`xRingbufferSendComplete()`](https://docs.espressif.com/projects/esp-idf/zh_CN/release-v5.2/esp32s3/api-reference/system/freertos_additions.html#_CPPv423xRingbufferSendComplete15RingbufHandle_tPv "xRingbufferSendComplete") 的文档。
- **可分割 buffer**：当数据项在 buffer 末尾绕回时，如果 buffer 头部和尾部的总空间足够，则支持将一个数据项分成两部分进行存储。可分割 buffer 比不可分割 buffer 更节省内存，但在检索时可能会返回数据项的两个部分。
- **字节 buffer**：不将数据存储为单独的数据项。所有数据都存储为字节序列，每次可以发送或检索任意大小的字节。当不需要单独维护数据项时，推荐使用字节 buffer，例如字节流。

> 不可分割 buffer 和可分割 buffer 在 `32` 位对齐地址上存储数据项。

- 因此，在检索一个数据项时，数据项指针一定也是 `32` 位对齐的。这在向 `DMA` 发送数据时非常有用。

> 存储在不可分割或可分割 buffer 中的每个数据项 **需要额外的 8 字节用于标头**。

- 数据项大小会向上取整为 32 位对齐大小，即 4 字节的倍数，实际的数据项大小则记录在标头中。不可分割和可分割 buffer 的大小在创建时也会向上取整。0
0x8000之前的地址保存Boot loader。

`Partition Table`分区：ESP32 在 flash 的 默认偏移地址 0x8000 处烧写一张分区表。该分区表的长度为 0xC00 字节（最多可以保存 95 条分区表条目）。分区表数据后还保存着该表的 MD5 校验和，用于验证分区表的完整性。此外，如果芯片使能了安全启动功能，则该分区表后还会保存签名信息。这个默认偏移地址在make menuconfig - Partition Table-offset of Partition Table这里设置。修改这个值，会影响到后面所有分区的偏移量。

`NVS`分区：这个实际上是NVS(2)分区。用于存储每台设备的 PHY 校准数据（注意，并不是 PHY 初始化数据）。也用于存储 Wi-Fi 数据（如果使用了 esp_wifi_set_storage(WIFI_STORAGE_FLASH) 初始化函数）和通过NVS API 保存的其他应用程序数据。强烈建议为 NVS 分区分配至少 0x3000 字节空间。

`OTA data分`区：系统从哪个app分区启动由此分区内的信息决定。

`Phy_init`分区：用于存储 PHY 初始化数据。这样可以为每个设备（而不是在固件中）配置 PHY。在默认的配置中，phy partition 未被使用。

`Factory app`分区：用于保存工厂（出厂）应用程序。如果分区表中有工厂应用程序，ESP-IDF 软件启动加载器会启动工厂应用程序。如果分区表中没有工厂应用程序，则启动第一个可用的 OTA 分区（通常是 OTA_0）。

`Core dump`分区：core dump分区用于查找系统崩溃时的软件错误，系统崩溃的时候会将调试信息写入到Flash中保存以便开发者对崩溃原因进行分析。关于这个分区的使用可以参考使用ESP32 的调试工具 coredump。

`OTA_0/OTA_1`分区用于保存OTA下载的固件。OTA启用后，OTA下载的固件镜像交替保存于OTA_0/OTA_1分区，镜像验证后，OTA data分区更新，指定在下一次启动时使用该镜像。OTA不会影响到Factory app分区，这样用户可以随时恢复到出厂状态。

`fctry`分区：保存阿里云四元组。如果不连接阿里云，该分区可以省略。

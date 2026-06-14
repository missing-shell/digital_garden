### EMMC 存储器分区表
| 名称         | 偏移  | 大小    | 文件系统  | 内容                                  |
| ---------- | --- | ----- | ----- | ----------------------------------- |
| bootloader | 33K | 8159K | RAW   | u-boot                              |
| boot       | 8M  | 64M   | FAT32 | Image, fsl-imx8mm-evk.dtb, logo.bmp |
| rootfs     | 72M | 7G+   | Ext4  | 文件系统                                |


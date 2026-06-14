# station
  * ESP32-C6 仅在 station 模式下支持 WPA2_Enterprise。
  * 通过函数 esp_netif_init() 初始化 lwIP 并创建一个 IwIP 任务，也称为 TCP/IP 任务。
  * lwIP 是 ESP-IDF 提供的 TCP/IP 库堆栈，用于执行 TCP、UDP、DHCP 等各种协议。
# 基本思路
## 初始化
### 初始化NVS
#### app_main
* 初始化NVS
* 初始化WiFi station[[#wifi_init_sta(void)]]
### 初始化LwIP
* [[softap#LwIP stack]]

## 无线网络配置
* 启动WiFi网络
### wifi_init_sta(void)
---
* 初始化Wi-Fi station
---
#### xEventGroupCreate()
* 创建一个事件组：同步WiFi连接的状态
```c++
s_wifi_event_group = xEventGroupCreate();
```
#### init
1. 初始化网络接口`esp_netif_init()`
2. 创建默认的事件循环[[#event loop]]
3. 创建默认的WiFi station
4. 初始化WiFi驱动
5. 注册事件处理器--处理WiFi和IP事件[[#event_handler()]]
6. 设置WiFi配置
7. 设置WiFi模式为station模式
8. 启动WiFi--开始WiFi通信
9. 等待连接到AP或连接失败的事件
### 建立连接
* 扫描阶段
* 验证阶段
* 关联阶段
* 四次握手阶段

## 处理WiFi事件

### event_handler()
* 处理Wi-Fi和IP事件，以便于正确地连接到AP。当连接成功时，它会设置一个事件位，当连接失败时，它会设置另一个事件位。
### event loop
* 接收 WiFi 和 TCP/IP 事件，例如当站点连接到 AP 时，或者当站点与 AP 断开连接时，或者当站点获取其 IP 地址时
* 允许应用程序任务注册一个回调，该回调将在 WiFi 或 TCP/IP 事件发生时执行。

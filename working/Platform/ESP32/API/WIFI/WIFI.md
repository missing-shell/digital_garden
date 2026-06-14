* IEEE 802.11是一种无线局域网标准，而Wi-Fi是IEEE 802.11标准的一种实现
* [Wi-Fi Direct](https://zh.wikipedia.org/wiki/Wi-Fi_Direct "Wi-Fi Direct")，直接进行文件传输和媒体共享。
* 2.4GHz频段是一种无须执照的频段，也就意味着，同样使用2.4GHz的电话、微波炉、蓝牙耳机等设备可能会争用该频段。
* 尽管最多支持 16 根天线，发送和接收数据时，最多仅能同时使能两根天线。API [`esp_wifi_set_ant()`](https://docs.espressif.com/projects/esp-idf/zh_CN/v5.1.1/esp32c6/api-reference/network/esp_wifi.html#_CPPv416esp_wifi_set_antPK17wifi_ant_config_t "esp_wifi_set_ant") 用于配置使能哪些天线。
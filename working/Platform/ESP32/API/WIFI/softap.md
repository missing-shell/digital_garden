* AP模式，也被称为接入点模式或者热点模式，是WiFi网络中的一种工作模式。在这种模式下，设备充当一个无线接入点，允许其他WiFi设备连接到它。这种模式通常用于创建一个无线局域网，或者将无线设备连接到有线网络。
* 在AP模式下，设备会创建一个SSID（服务集标识符），其他设备可以搜索到这个SSID并请求连接。连接请求需要通过认证和关联两个步骤。认证步骤主要是验证请求设备是否有权连接到网络，关联步骤则是建立设备之间的通信。一旦设备连接到AP，它们就可以通过AP与网络中的其他设备进行通信
* 使能 Wi-Fi NVS 时，所有配置都将存储到 flash 中。
# Application
## app_task

### app_main()
 * 初始化NVS(非易失性存储)
 * 调用[[#wifi_init_softap()]]初始化WiFi SoftAP
### wifi_init_softap()
***
* **用于初始化esp32作为软件接入点(softap) 的WiFi**
* ***
1. 初始化网络接口和[[#esp_event_loop_create_default()]]事件循环---esp-idf网络编程的基础步骤，用于设置网络接口和事件处理系统
2. 创建默认的WiFi AP
3. 初始化WiFi驱动
4. 注册事件处理器--[[#wifi_event_handler()]]
5. 设置WiFi配置--SSID password channel max_connection mode等--如果密码长度为零，认证模式将设置为开放模式
6. 设置WiFi模式为AP
7. 应用WiFi配置
8. 启动WiFi配置--开始广播SSID，允许设备连接
9. 打印WiFi信息--SSID password channel
### 获取esp32 ap 的IP地址
### 更改esp32 ap的IP地址
# Event tasks
### wifi_event_handler()
* 处理WiFi时间，当有设备连接或断开连接时，打印相关信息
### esp_event_loop_create_default()
* 一个无限循环，用于接收和处理系统中发生的各种事件，例如 WiFi 连接、断开连接、接收到数据等。每个事件都会被放入一个队列中，然后由事件循环依次取出并处理。
### code
#### 创建默认事件循环
```c++
	ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    esp_netif_create_default_wifi_ap();
```
#### 注册事件处理器
```c++
ESP_ERROR_CHECK(esp_event_handler_instance_register(WIFI_EVENT,
                                                        ESP_EVENT_ANY_ID,
                                                        &wifi_event_handler,
                                                        NULL,
                                                        NULL));
```
#### 在事件处理器中处理事件
```c++
static void wifi_event_handler(void *arg, esp_event_base_t event_base,
                               int32_t event_id, void *event_data)
{
    if (event_id == WIFI_EVENT_AP_STACONNECTED)
    {
        wifi_event_ap_staconnected_t *event = (wifi_event_ap_staconnected_t *)event_data;
        ESP_LOGI(TAG, "station " MACSTR " join, AID=%d",
                 MAC2STR(event->mac), event->aid);
    }
    else if (event_id == WIFI_EVENT_AP_STADISCONNECTED)
    {
        wifi_event_ap_stadisconnected_t *event = (wifi_event_ap_stadisconnected_t *)event_data;
        ESP_LOGI(TAG, "station " MACSTR " leave, AID=%d",
                 MAC2STR(event->mac), event->aid);
    }
}
```

# LwIP stack
* LwIP stack 是一个轻量级的 IP 堆栈，用于嵌入式系统。
* lwIP实现了TCP,UDP,IP,DHCP等的协议。
* 将来可以用另一个 TCP/IP 库替换 lwIP，并且无需修改 ESP32 应用程序代码。ESP-NETIF 也是线程安全的。
* **esp_netif_init()** 函数执行 lwIP 初始化并创建 lwIP 任务。
## esp-idf wifi 编程模型
![[esp-idf WiFi编程模型.png]]
# 代码流程图[[WIFI_AP.canvas|WIFI_AP]]
#### code
```c++
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_mac.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"

#include "lwip/err.h"
#include "lwip/sys.h"

/* The examples use WiFi configuration that you can set via project configuration menu.

   If you'd rather not, just change the below entries to strings with
   the config you want - ie #define EXAMPLE_WIFI_SSID "mywifissid"
*/
#define EXAMPLE_ESP_WIFI_SSID "你好"
#define EXAMPLE_ESP_WIFI_PASS "12345678"
#define EXAMPLE_ESP_WIFI_CHANNEL CONFIG_ESP_WIFI_CHANNEL
#define EXAMPLE_MAX_STA_CONN CONFIG_ESP_MAX_STA_CONN

static const char *TAG = "wifi softAP";

static void wifi_event_handler(void *arg, esp_event_base_t event_base,
                               int32_t event_id, void *event_data)
{
    if (event_id == WIFI_EVENT_AP_STACONNECTED)
    {
        wifi_event_ap_staconnected_t *event = (wifi_event_ap_staconnected_t *)event_data;
        ESP_LOGI(TAG, "station " MACSTR " join, AID=%d",
                 MAC2STR(event->mac), event->aid);
    }
    else if (event_id == WIFI_EVENT_AP_STADISCONNECTED)
    {
        wifi_event_ap_stadisconnected_t *event = (wifi_event_ap_stadisconnected_t *)event_data;
        ESP_LOGI(TAG, "station " MACSTR " leave, AID=%d",
                 MAC2STR(event->mac), event->aid);
    }
}

void wifi_init_softap(void)
{
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    esp_netif_create_default_wifi_ap();

    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));

    ESP_ERROR_CHECK(esp_event_handler_instance_register(WIFI_EVENT,
                                                        ESP_EVENT_ANY_ID,
                                                        &wifi_event_handler,
                                                        NULL,
                                                        NULL));

    wifi_config_t wifi_config = {
        .ap = {
            .ssid = EXAMPLE_ESP_WIFI_SSID,
            .ssid_len = strlen(EXAMPLE_ESP_WIFI_SSID),
            .channel = EXAMPLE_ESP_WIFI_CHANNEL,
            .password = EXAMPLE_ESP_WIFI_PASS,
            .max_connection = EXAMPLE_MAX_STA_CONN,
#ifdef CONFIG_ESP_WIFI_SOFTAP_SAE_SUPPORT
            .authmode = WIFI_AUTH_WPA3_PSK,
            .sae_pwe_h2e = WPA3_SAE_PWE_BOTH,
#else /* CONFIG_ESP_WIFI_SOFTAP_SAE_SUPPORT */
            .authmode = WIFI_AUTH_WPA2_PSK,
#endif
            .pmf_cfg = {
                .required = true,
            },
        },
    };
    if (strlen(EXAMPLE_ESP_WIFI_PASS) == 0)
    {
        wifi_config.ap.authmode = WIFI_AUTH_OPEN;
    }

    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_AP));
    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_AP, &wifi_config));
    ESP_ERROR_CHECK(esp_wifi_start());

    ESP_LOGI(TAG, "wifi_init_softap finished. SSID:%s password:%s channel:%d",
             EXAMPLE_ESP_WIFI_SSID, EXAMPLE_ESP_WIFI_PASS, EXAMPLE_ESP_WIFI_CHANNEL);
}

void app_main(void)
{
    // Initialize NVS
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND)
    {
        ESP_ERROR_CHECK(nvs_flash_erase());
        ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);

    ESP_LOGI(TAG, "ESP_WIFI_MODE_AP");
    wifi_init_softap();
}

```
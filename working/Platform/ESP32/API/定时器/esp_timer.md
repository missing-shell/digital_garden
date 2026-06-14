 ```c++
 #include <stdio.h>     // 标准输入输出库
#include <string.h>    // 字符串处理库
#include <unistd.h>    // UNIX 标准库
#include "esp_timer.h" // ESP32 定时器库
#include "esp_log.h"   // ESP32 日志库
#include "esp_sleep.h" // ESP32 睡眠库
#include "sdkconfig.h" // ESP32 配置文件

static void periodic_timer_callback(void *arg); // 周期性定时器回调函数
static void oneshot_timer_callback(void *arg);  // 单次定时器回调函数

static const char *TAG = "example"; // 日志标签


    /* 创建两个定时器：
     * 1. 周期性定时器，每0.5秒运行一次，并打印一条消息
     * 2. 单次定时器，5秒后触发，并重新以1秒的周期启动周期性定时器
     */

    const esp_timer_create_args_t periodic_timer_args = {
        .callback = &periodic_timer_callback, // 周期性定时器回调函数
        /* 名称是可选的，但在调试时可能有帮助 */
        .name = "periodic"};

    esp_timer_handle_t periodic_timer;
    ESP_ERROR_CHECK(esp_timer_create(&periodic_timer_args, &periodic_timer)); // 创建周期性定时器
    /* 定时器已创建，但尚未启动 */

    const esp_timer_create_args_t oneshot_timer_args = {
        .callback = &oneshot_timer_callback, // 单次定时器回调函数
        /* 在此处指定的参数将传递给定时器回调函数 */
        .arg = (void *)periodic_timer,
        .name = "one-shot"};
    esp_timer_handle_t oneshot_timer;
    ESP_ERROR_CHECK(esp_timer_create(&oneshot_timer_args, &oneshot_timer)); // 创建单次定时器

    /* 启动定时器 */
    ESP_ERROR_CHECK(esp_timer_start_periodic(periodic_timer, 500000)); // 启动周期性定时器，周期为0.5秒
    ESP_ERROR_CHECK(esp_timer_start_once(oneshot_timer, 5000000));     // 启动单次定时器，延迟5秒触发
    ESP_LOGI(TAG, "已启动定时器，自开机以来的时间：%lld 微秒", esp_timer_get_time());

    /* 每2秒在控制台打印有关定时器的调试信息 */
    for (int i = 0; i < 5; ++i)
    {
        ESP_ERROR_CHECK(esp_timer_dump(stdout)); // 打印定时器调试信息
        usleep(2000000);                         // 延迟2秒
    }

    /* 在轻度睡眠中继续计时，并在轻度睡眠后正确调度定时器 */
    int64_t t1 = esp_timer_get_time();
    ESP_LOGI(TAG, "进入0.5秒的轻度睡眠，自开机以来的时间：%lld 微秒", t1);

    ESP_ERROR_CHECK(esp_sleep_enable_timer_wakeup(500000)); // 设置轻度睡眠的唤醒时间为0.5秒
    esp_light_sleep_start();                                // 进入轻度睡眠

    int64_t t2 = esp_timer_get_time();
    ESP_LOGI(TAG, "从轻度睡眠中唤醒，自开机以来的时间：%lld 微秒", t2);

    assert(llabs((t2 - t1) - 500000) < 1000); // 检查轻度睡眠的唤醒时间是否正确

    /* 让定时器再运行一段时间 */
    usleep(2000000); // 延迟2秒

    /* 清理并完成示例 */
    ESP_ERROR_CHECK(esp_timer_stop(periodic_timer));   // 停止周期性定时器
    ESP_ERROR_CHECK(esp_timer_delete(periodic_timer)); // 删除周期性定时器
    ESP_ERROR_CHECK(esp_timer_delete(oneshot_timer));  // 删除单次定时器
    ESP_LOGI(TAG, "已停止并删除定时器");
}

static void periodic_timer_callback(void *arg)
{
    int64_t time_since_boot = esp_timer_get_time();
    ESP_LOGI(TAG, "周期性定时器被调用，自开机以来的时间：%lld 微秒", time_since_boot);
}

static void oneshot_timer_callback(void *arg)
{
    int64_t time_since_boot = esp_timer_get_time();
    ESP_LOGI(TAG, "单次定时器被调用，自开机以来的时间：%lld 微秒", time_since_boot);
    esp_timer_handle_t periodic_timer_handle = (esp_timer_handle_t)arg;
    /* 要启动正在运行的定时器，需要先停止它 */
    ESP_ERROR_CHECK(esp_timer_stop(periodic_timer_handle));                    // 停止周期性定时器
    ESP_ERROR_CHECK(esp_timer_start_periodic(periodic_timer_handle, 1000000)); // 以1秒的周期重新启动周期性定时器
    time_since_boot = esp_timer_get_time();
    ESP_LOGI(TAG, "以1秒的周期重新启动周期性定时器，自开机以来的时间：%lld 微秒",
             time_since_boot);
}
```
```C++
#include <stdio.h>
#include "driver/gpio.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#define LED_PIN GPIO_NUM_18
#define SIMULATED_BUTTON_PIN GPIO_NUM_19

int led_state = 0;  // 全局变量，用于跟踪LED状态

void button_isr_handler(void* arg)
{
    led_state = !led_state;  // 切换LED状态
    gpio_set_level(LED_PIN, led_state);  // 设置LED引脚电平为当前状态
    vTaskDelay(pdMS_TO_TICKS(1000));  // 延迟1秒
}

void app_main(void)
{
    gpio_set_level(LED_PIN, 1);  // 设置LED引脚电平为高，使灯泡亮起

    // 安装GPIO中断服务
    gpio_install_isr_service(0);

    // 配置按键引脚
    gpio_config_t button_config = {
        .pin_bit_mask = (1ULL << SIMULATED_BUTTON_PIN),  // 按键引脚位掩码
        .mode = GPIO_MODE_INPUT,  // 输入模式
        .pull_up_en = GPIO_PULLUP_ENABLE,  // 启用上拉电阻
        .intr_type = GPIO_INTR_ANYEDGE  // 任意边沿触发中断
    };
    gpio_config(&button_config);

    // 配置LED引脚
    gpio_config_t ioconfig = {
        .pin_bit_mask = (1ULL << LED_PIN),  // LED引脚位掩码
        .mode = GPIO_MODE_OUTPUT  // 输出模式
    };
    gpio_config(&ioconfig);

    // 注册按键中断处理函数
    gpio_isr_handler_add(SIMULATED_BUTTON_PIN, button_isr_handler, NULL);

    while (1)
    {
        button_isr_handler(NULL);  // 模拟按键按下，调用中断处理函数
        vTaskDelay(pdMS_TO_TICKS(1000));  // 延迟1秒
    }
}

```
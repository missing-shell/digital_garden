```c++
#include <stdio.h>  // 包含标准输入/输出库
#include <string.h>  // 包含字符串处理函数库
#include <stdlib.h>  // 包含一些常用函数库
#include <inttypes.h>  // 包含定长整型的定义
#include "freertos/FreeRTOS.h"  // 包含FreeRTOS的基础头文件
#include "freertos/task.h"  // 包含FreeRTOS的任务处理头文件
#include "freertos/queue.h"  // 包含FreeRTOS的队列处理头文件
#include "driver/gpio.h"  // 包含ESP32的GPIO驱动头文件

#define GPIO_OUTPUT_IO_0    CONFIG_GPIO_OUTPUT_0  // 定义GPIO输出0的引脚号
#define GPIO_OUTPUT_IO_1    CONFIG_GPIO_OUTPUT_1  // 定义GPIO输出1的引脚号
#define GPIO_OUTPUT_PIN_SEL  ((1ULL<<GPIO_OUTPUT_IO_0) | (1ULL<<GPIO_OUTPUT_IO_1))  // 定义GPIO输出引脚的选择
#define GPIO_INPUT_IO_0     CONFIG_GPIO_INPUT_0  //定义GPIO输入0的引脚号
#define GPIO_INPUT_IO_1     CONFIG_GPIO_INPUT_1  //定义GPIO输入1的引脚号
#define GPIO_INPUT_PIN_SEL  ((1ULL<<GPIO_INPUT_IO_0) | (1ULL<<GPIO_INPUT_IO_1))  //定义GPIO输入引脚的选择
#define ESP_INTR_FLAG_DEFAULT 0  // 设置中断的默认标志

static QueueHandle_t gpio_evt_queue = NULL;  // 定义一个句柄，用来指向队列

static void IRAM_ATTR gpio_isr_handler(void* arg)  //定义一个中断服务程序处理器
{
    uint32_t gpio_num = (uint32_t) arg; //将传入的参数转为无符号整型
    xQueueSendFromISR(gpio_evt_queue, &gpio_num, NULL);  //从中断服务程序发送消息到队列
}

static void gpio_task_example(void* arg)  //定义一个GPIO任务
{
    uint32_t io_num;  //定义一个无符号整型变量
    for(;;) {  //无限循环，进行任务处理
        if(xQueueReceive(gpio_evt_queue, &io_num, portMAX_DELAY)) {  //如果从队列中接收到消息
            printf("GPIO[%"PRIu32"] intr, val: %d\n", io_num, gpio_get_level(io_num));  //打印GPIO的值
        }
    }
}

void app_main(void)  //程序主体部分：配置GPIO，创建任务，处理任务
{
    gpio_config_t io_conf = {};  //定义一个GPIO配置结构体，并进行初始化
    io_conf.intr_type = GPIO_INTR_DISABLE;  // 设置为禁用中断
    io_conf.mode = GPIO_MODE_OUTPUT;  // 设置为输出模式
    io_conf.pin_bit_mask = GPIO_OUTPUT_PIN_SEL;  // 设置需要设置的GPIO
    io_conf.pull_down_en = 0;   // 禁用下拉模式
    io_conf.pull_up_en = 0;   //禁用上拉模式
    gpio_config(&io_conf);  //使用以上参数配置GPIO

    io_conf.intr_type = GPIO_INTR_POSEDGE;  // 设置为上升沿触发的中断
    io_conf.pin_bit_mask = GPIO_INPUT_PIN_SEL;  //设置需要设置的GPIO
    io_conf.mode = GPIO_MODE_INPUT;  //设置为输入模式
    io_conf.pull_up_en = 1;  //开启上拉模式
    gpio_config(&io_conf);   // 使用以上参数配置GPIO

    gpio_set_intr_type(GPIO_INPUT_IO_0, GPIO_INTR_ANYEDGE);  // 设置GPIO的中断类型，此处设置为任何边沿触发中断

    gpio_evt_queue = xQueueCreate(10, sizeof(uint32_t));  // 创建一个新队列，用于处理GPIO事件
    xTaskCreate(gpio_task_example, "gpio_task_example", 2048, NULL, 10, NULL);  // 创建一个新任务，用于处理GPIO事件

    gpio_install_isr_service(ESP_INTR_FLAG_DEFAULT);  // 安装GPIO的中断服务程序
    gpio_isr_handler_add(GPIO_INPUT_IO_0, gpio_isr_handler, (void*) GPIO_INPUT_IO_0);  // 为特定的GPIO引脚添加中断服务程序处理器
    gpio_isr_handler_add(GPIO_INPUT_IO_1, gpio_isr_handler, (void*) GPIO_INPUT_IO_1);  // 为特定的GPIO引脚添加中断服务程序处理器

    gpio_isr_handler_remove(GPIO_INPUT_IO_0);  // 删除特定GPIO的中断服务程序处理器
    gpio_isr_handler_add(GPIO_INPUT_IO_0, gpio_isr_handler, (void*) GPIO_INPUT_IO_0);  // 为特定的GPIO引脚再次添加中断服务程序处理器

    printf("Minimum free heap size: %"PRIu32" bytes\n", esp_get_minimum_free_heap_size());  // 打印最小空闲堆大小

    int cnt = 0;
    while(1) {  //无限循环，进行任务处理
        printf("cnt: %d\n", cnt++);  //打印并增加计数器
        vTaskDelay(1000 / portTICK_PERIOD_MS);  // 触发任务延时，延时1秒（1000毫秒）
        gpio_set_level(GPIO_OUTPUT_IO_0, cnt % 2);  // 设置GPIO的输出为0或1
        gpio_set_level(GPIO_OUTPUT_IO_1, cnt % 2);  // 设置GPIO的输出为0或1
    }
}

```
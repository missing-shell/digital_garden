```c++

#include <string.h>  // 包含字符串处理函数库
#include <stdio.h>   // 包含输入/输出函数库
#include "sdkconfig.h"  // 包含SDK配置
#include "esp_log.h"  // 包含ESP log处理
#include "freertos/FreeRTOS.h"  // 包含FreeRTOS库
#include "freertos/task.h"  // 包含FreeRTOS任务处理
#include "freertos/semphr.h"  // 包含FreeRTOS信号量处理
#include "esp_adc/adc_continuous.h"  // 包含ESP ADC连续读取处理

#define EXAMPLE_ADC_UNIT                    ADC_UNIT_1  // 定义示例ADC单元
#define _EXAMPLE_ADC_UNIT_STR(unit)         #unit   // 定义示例ADC单元字符串
#define EXAMPLE_ADC_UNIT_STR(unit)          _EXAMPLE_ADC_UNIT_STR(unit)  //定义示例ADC单元变量
#define EXAMPLE_ADC_CONV_MODE               ADC_CONV_SINGLE_UNIT_1  // 定义示例ADC转换模式
#define EXAMPLE_ADC_ATTEN                   ADC_ATTEN_DB_0  // 定义示例ADC衰减
#define EXAMPLE_ADC_BIT_WIDTH               SOC_ADC_DIGI_MAX_BITWIDTH  // 定义示例ADC位宽

#if CONFIG_IDF_TARGET_ESP32 || CONFIG_IDF_TARGET_ESP32S2  // 如果目标是ESP32或ESP32S2
#define EXAMPLE_ADC_OUTPUT_TYPE             ADC_DIGI_OUTPUT_FORMAT_TYPE1  // 定义示例ADC输出类型
#define EXAMPLE_ADC_GET_CHANNEL(p_data)     ((p_data)->type1.channel) // 定义获取示例ADC通道的宏
#define EXAMPLE_ADC_GET_DATA(p_data)        ((p_data)->type1.data)   // 定义获取示例ADC数据的宏
#else  // 否则
#define EXAMPLE_ADC_OUTPUT_TYPE             ADC_DIGI_OUTPUT_FORMAT_TYPE2  // 定义示例ADC输出类型
#define EXAMPLE_ADC_GET_CHANNEL(p_data)     ((p_data)->type2.channel) // 定义获取示例ADC通道的宏
#define EXAMPLE_ADC_GET_DATA(p_data)        ((p_data)->type2.data)   // 定义获取示例ADC数据的宏
#endif

#define EXAMPLE_READ_LEN                    256  // 定义示例读取长度

#if CONFIG_IDF_TARGET_ESP32  // 如果目标是ESP32
static adc_channel_t channel[2] = {ADC_CHANNEL_6, ADC_CHANNEL_7};  // 定义静态 ADC通道数组
#else   // 否则
static adc_channel_t channel[2] = {ADC_CHANNEL_2, ADC_CHANNEL_3};  // 定义静态 ADC通道数组
#endif

static TaskHandle_t s_task_handle;  // 定义静态任务句柄
static const char *TAG = "EXAMPLE";  // 定义静态标签

static bool IRAM_ATTR s_conv_done_cb(adc_continuous_handle_t handle, const adc_continuous_evt_data_t *edata, void *user_data)
{   // 定义在ADC连续转换完成后的回调函数
    BaseType_t mustYield = pdFALSE;  // 定义必须让步的基本类型
    //Notify that ADC continuous driver has done enough number of conversions  // 通知ADC连续驱动足够多的转换已完成
    vTaskNotifyGiveFromISR(s_task_handle, &mustYield);  // 从中断服务例程给出任务通知

    return (mustYield == pdTRUE);  // 如果必须让步，则返回真
}

static void continuous_adc_init(adc_channel_t *channel, uint8_t channel_num, adc_continuous_handle_t *out_handle)
{  // 定义连续 ADC初始化函数
    adc_continuous_handle_t handle = NULL;  //定义 ADC连续句柄

    adc_continuous_handle_cfg_t adc_config = {  // 定义 ADC连续配置
        .max_store_buf_size = 1024,  // 最大储存缓冲区大小
        .conv_frame_size = EXAMPLE_READ_LEN,  //转换帧大小
    };
    ESP_ERROR_CHECK(adc_continuous_new_handle(&adc_config, &handle));  // 检查创建新的ADC连续句柄

    adc_continuous_config_t dig_cfg = {  // 定义 ADC连续配置
        .sample_freq_hz = 20 * 1000,  //样本频率
        .conv_mode = EXAMPLE_ADC_CONV_MODE,  // 转换模式
        .format = EXAMPLE_ADC_OUTPUT_TYPE,  // 格式
    };

    adc_digi_pattern_config_t adc_pattern[SOC_ADC_PATT_LEN_MAX] = {0};  // 定义 ADC数字模式配置
    dig_cfg.pattern_num = channel_num;  // 设定模式数量
    for (int i = 0; i < channel_num; i++) {  // 对每个通道进行循环
        adc_pattern[i].atten = EXAMPLE_ADC_ATTEN;  // 设定衰减
        adc_pattern[i].channel = channel[i] & 0x7;  // 设定通道
        adc_pattern[i].unit = EXAMPLE_ADC_UNIT;  //设定单位
        adc_pattern[i].bit_width = EXAMPLE_ADC_BIT_WIDTH;  // 设定位宽

        ESP_LOGI(TAG, "adc_pattern[%d].atten is :%"PRIx8, i, adc_pattern[i].atten);  // 记录ADC模式的衰减信息
        ESP_LOGI(TAG, "adc_pattern[%d].channel is :%"PRIx8, i, adc_pattern[i].channel);  // 记录ADC模式的通道信息
        ESP_LOGI(TAG, "adc_pattern[%d].unit is :%"PRIx8, i, adc_pattern[i].unit);  //记录ADC模式的单位信息
    }
    dig_cfg.adc_pattern = adc_pattern;  // 设定ADC模式
    ESP_ERROR_CHECK(adc_continuous_config(handle, &dig_cfg));  // 检查ADC连续配置

    *out_handle = handle;  // 设置输出句柄
}

void app_main(void) //主程序入口
{
    esp_err_t ret;  //ESP错误类型
    uint32_t ret_num = 0;  //返回数量
    uint8_t result[EXAMPLE_READ_LEN] = {0};  //结果数组
    memset(result, 0xcc, EXAMPLE_READ_LEN);  //将结果数组所有元素初始化为0xcc

    s_task_handle = xTaskGetCurrentTaskHandle();  //获取当前任务句柄

    adc_continuous_handle_t handle = NULL;  //ADC连续句柄
    continuous_adc_init(channel, sizeof(channel) / sizeof(adc_channel_t), &handle);  //初始化ADC连续转换

    adc_continuous_evt_cbs_t cbs = {
        .on_conv_done = s_conv_done_cb,  //设置转换完成回调
    };
    ESP_ERROR_CHECK(adc_continuous_register_event_callbacks(handle, &cbs, NULL));  //检查ADC连续注册事件回调的结果
    ESP_ERROR_CHECK(adc_continuous_start(handle));  //检查ADC连续开始的结果

    while(1) {  //无限循环，主程序在此运行

        // 等待任务通知
        ulTaskNotifyTake(pdTRUE, portMAX_DELAY);

        char unit[] = EXAMPLE_ADC_UNIT_STR(EXAMPLE_ADC_UNIT);  //单元名称字符串

        while (1) {  //嵌套循环
            ret = adc_continuous_read(handle, result, EXAMPLE_READ_LEN, &ret_num, 0);  //读取ADC连续值
            if (ret == ESP_OK) {  //如果读取成功
                ESP_LOGI("TASK", "ret is %x, ret_num is %"PRIu32" bytes", ret, ret_num);  //打印返回值和返回数量
                for (int i = 0; i < ret_num; i += SOC_ADC_DIGI_RESULT_BYTES) {  //遍历结果数组
                    adc_digi_output_data_t *p = (void*)&result[i];
                    uint32_t chan_num = EXAMPLE_ADC_GET_CHANNEL(p);  //获取通道
                    uint32_t data = EXAMPLE_ADC_GET_DATA(p);  //获取数据

                    // 如果通道号小于单位的最大通道数量，则数据有效
                    if (chan_num < SOC_ADC_CHANNEL_NUM(EXAMPLE_ADC_UNIT)) {
                        ESP_LOGI(TAG, "Unit: %s, Channel: %"PRIu32", Value: %"PRIx32, unit, chan_num, data);  //打印单位、通道和值
                    } else {  //否则，数据无效
                        ESP_LOGW(TAG, "Invalid data [%s_%"PRIu32"_%"PRIx32"]", unit, chan_num, data);  //打印无效数据
                    }
                }
                // delay to avoid task watchdog timeout
                vTaskDelay(1);  //延迟1ms
            } else if (ret == ESP_ERR_TIMEOUT) {//如果读取超时
                // 读取结果直到API返回超时，这意味着没有可用数据
                break;  //跳出嵌套循环
            }
        }
    }

    ESP_ERROR_CHECK(adc_continuous_stop(handle));  //检查停止ADC连续转换的结果
    ESP_ERROR_CHECK(adc_continuous_deinit(handle));  //检查取消初始化ADC连续转换的结果
 
}

```

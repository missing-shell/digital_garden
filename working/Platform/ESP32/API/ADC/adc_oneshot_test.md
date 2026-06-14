 ```c++
 
 #include <stdio.h> // 包含标准输入输出库
#include <stdlib.h> // 包含标准库
#include <string.h> // 包含字符串处理库
#include "freertos/FreeRTOS.h" // 包含FreeRTOS实时操作系统库
#include "freertos/task.h" // 包含FreeRTOS任务处理库
#include "soc/soc_caps.h" // 包含SoC功能库
#include "esp_log.h" // 包含ESP日志库
#include "esp_adc/adc_oneshot.h" // 包含ESP ADC一次性读取库
#include "esp_adc/adc_cali.h" // 包含ESP ADC校准库
#include "esp_adc/adc_cali_scheme.h" // 包含ESP ADC校准方案库
 
const static char *TAG = "EXAMPLE"; // 定义日志标签

#define EXAMPLE_ADC1_CHAN0 ADC_CHANNEL_0 // 定义ADC1通道0
#define EXAMPLE_ADC1_CHAN1 ADC_CHANNEL_1 // 定义ADC1通道1

#define EXAMPLE_ADC_ATTEN ADC_ATTEN_DB_11 // 定义ADC衰减值

static int adc_raw[2][10]; // 定义ADC原始数据数组
static int voltage[2][10]; // 定义电压数据数组
static bool example_adc_calibration_init(adc_unit_t unit, adc_channel_t channel, adc_atten_t atten, adc_cali_handle_t *out_handle); // 定义ADC校准初始化函数
static void example_adc_calibration_deinit(adc_cali_handle_t handle); // 定义ADC校准终止函数

void app_main() // 主函数
{
    // ADC1初始化
    adc_oneshot_unit_handle_t adc1_handle; // 定义ADC1句柄
    adc_oneshot_unit_init_cfg_t init_config1 = { // 定义ADC1初始化配置
        .unit_id = ADC_UNIT_1, // 设置ADC单元ID为1
        .ulp_mode = ADC_ULP_MODE_DISABLE, // 禁用ULP模式
    };
    ESP_ERROR_CHECK(adc_oneshot_new_unit(&init_config1, &adc1_handle)); // 创建新的ADC1单元

    // ADC1配置
    adc_oneshot_chan_cfg_t config = { // 定义ADC1通道配置
        .bitwidth = ADC_BITWIDTH_DEFAULT, // 设置位宽为默认值
        .atten = EXAMPLE_ADC_ATTEN, // 设置衰减值
    };
    ESP_ERROR_CHECK(adc_oneshot_config_channel(adc1_handle, EXAMPLE_ADC1_CHAN0, &config)); // 配置ADC1通道0
    ESP_ERROR_CHECK(adc_oneshot_config_channel(adc1_handle, EXAMPLE_ADC1_CHAN1, &config)); // 配置ADC1通道1

    // ADC1校准初始化
    adc_cali_handle_t adc1_cali_chan0_handle = NULL; // 定义ADC1通道0校准句柄
    adc_cali_handle_t adc1_cali_chan1_handle = NULL; // 定义ADC1通道1校准句柄
    bool do_calibration1_chan0 = example_adc_calibration_init(ADC_UNIT_1, EXAMPLE_ADC1_CHAN0, EXAMPLE_ADC_ATTEN, &adc1_cali_chan0_handle); // 初始化ADC1通道0校准
    bool do_calibration1_chan1 = example_adc_calibration_init(ADC_UNIT_1, EXAMPLE_ADC1_CHAN1, EXAMPLE_ADC_ATTEN, &adc1_cali_chan1_handle); // 初始化ADC1通道1校准

    while (1) // 无限循环
    {
        ESP_ERROR_CHECK(adc_oneshot_read(adc1_handle, EXAMPLE_ADC1_CHAN0, &adc_raw[0][0])); // 读取ADC1通道0的原始数据
        ESP_LOGI(TAG, "ADC%d通道[%d]原始数据： %d", ADC_UNIT_1 + 1, EXAMPLE_ADC1_CHAN0, adc_raw[0][0]); // 打印ADC1通道0的原始数据

        if (do_calibration1_chan0) // 如果进行了ADC1通道0的校准
        {
            ESP_ERROR_CHECK(adc_cali_raw_to_voltage(adc1_cali_chan0_handle, adc_raw[0][0], voltage[0][0])); // 将ADC1通道0的原始数据转换为电压值
            ESP_LOGI(TAG, "ADC%d通道[%d]校准电压: %d mV", ADC_UNIT_1 + 1, EXAMPLE_ADC1_CHAN0, voltage[0][0]); // 打印ADC1通道0的校准电压值
        }
        vTaskDelay(pdMS_TO_TICKS(1000)); // 延迟1秒

        ESP_ERROR_CHECK(adc_oneshot_read(adc1_handle, EXAMPLE_ADC1_CHAN1, &adc_raw[0][1])); // 读取ADC1通道1的原始数据
        ESP_LOGI(TAG, "ADC%d通道[%d]原始数据： %d", ADC_UNIT_1 + 1, EXAMPLE_ADC1_CHAN1, adc_raw[0][1]); // 打印ADC1通道1的原始数据

        if (do_calibration1_chan1) // 如果进行了ADC1通道1的校准
        {
            ESP_ERROR_CHECK(adc_cali_raw_to_voltage(adc1_cali_chan1_handle, adc_raw[0][1], voltage[0][1])); // 将ADC1通道1的原始数据转换为电压值
            ESP_LOGI(TAG, "ADC%d通道[%d]校准电压: %d mV", ADC_UNIT_1 + 1, EXAMPLE_ADC1_CHAN1, voltage[0][1]); // 打印ADC1通道1的校准电压值
        }
        vTaskDelay(pdMS_TO_TICKS(1000)); // 延迟1秒
    }

    // 终止
    ESP_ERROR_CHECK(adc_oneshot_del_unit(adc1_handle)); // 删除ADC1单元
    if (do_calibration1_chan0) // 如果进行了ADC1通道0的校准
    {
        example_adc_calibration_deinit(adc1_cali_chan0_handle); // 终止ADC1通道0的校准
    }
    if (do_calibration1_chan1) // 如果进行了ADC1通道1的校准
    {
        example_adc_calibration_deinit(adc1_cali_chan1_handle); // 终止ADC1通道1的校准
    }
}


// 初始化ADC校准
static bool example_adc_calibration_init(adc_unit_t unit, adc_channel_t channel, adc_atten_t atten, adc_cali_handle_t *out_handle)
{
    adc_cali_handle_t handle = NULL; // 定义校准句柄
    esp_err_t ret = ESP_FAIL; // 定义返回值
    bool calibrated = false; // 定义是否校准标志

#if ADC_CALI_SCHEME_CURVE_FITTING_SUPPORTED // 如果支持曲线拟合校准方案
    // 曲线拟合校准方案
    if (!calibrated) // 如果还未校准
    {
        ESP_LOGI(TAG, "校准方案版本：%s", "曲线拟合"); // 打印校准方案版本
        adc_cali_curve_fitting_config_t cali_config = { // 定义曲线拟合校准配置
            .unit_id = unit, // 设置ADC单元ID
            .chan = channel, // 设置ADC通道
            .atten = atten, // 设置衰减值
            .bitwidth = ADC_BITWIDTH_DEFAULT, // 设置位宽为默认值
        };
        ret = adc_cali_create_scheme_curve_fitting(&cali_config, &handle); // 创建曲线拟合校准方案
        if (ret == ESP_OK) // 如果创建成功
        {
            calibrated = true; // 设置已校准标志
        }
    }
#endif

#if ADC_CALI_SCHEME_LINE_FITTING_SUPPORTED // 如果支持线性拟合校准方案
    // 线性拟合校准方案
    if (!calibrated) // 如果还未校准
    {
        ESP_LOGI(TAG, "校准方案版本：%s", "线性拟合"); // 打印校准方案版本
        adc_cali_line_fitting_config_t cali_config = { // 定义线性拟合校准配置
            .unit_id = unit, // 设置ADC单元ID
            .atten = atten, // 设置衰减值
            .bitwidth = ADC_BITWIDTH_DEFAULT, // 设置位宽为默认值
        };
        ret = adc_cali_create_scheme_line_fitting(&cali_config, &handle); // 创建线性拟合校准方案
        if (ret == ESP_OK) // 如果创建成功
        {
            calibrated = true; // 设置已校准标志
        }
    }
#endif

    *out_handle = handle; // 输出校准句柄
    if (ret == ESP_OK) // 如果校准成功
    {
        ESP_LOGI(TAG, "校准成功"); // 打印校准成功
    }
    else if (ret == ESP_ERR_NOT_SUPPORTED || !calibrated) // 如果不支持校准或未校准
    {
        ESP_LOGW(TAG, "eFuse未烧录，跳过软件校准"); // 打印警告信息
    }
    else // 如果其他错误
    {
        ESP_LOGE(TAG, "无效参数或无内存"); // 打印错误信息
    }

    return calibrated; // 返回是否校准标志
}

// 停止ADC校准
static void example_adc_calibration_deinit(adc_cali_handle_t handle)
{
#if ADC_CALI_SCHEME_CURVE_FITTING_SUPPORTED // 如果支持曲线拟合校准方案
    ESP_LOGI(TAG, "注销%s校准方案", "曲线拟合"); // 打印注销校准方案信息
    ESP_ERROR_CHECK(adc_cali_delete_scheme_curve_fitting(handle)); // 删除曲线拟合校准方案

#elif ADC_CALI_SCHEME_LINE_FITTING_SUPPORTED // 如果支持线性拟合校准方案
    ESP_LOGI(TAG, "注销%s校准方案", "线性拟合"); // 打印注销校准方案信息
    ESP_ERROR_CHECK(adc_cali_delete_scheme_line_fitting(handle)); // 删除线性拟合校准方案
#endif
}

```
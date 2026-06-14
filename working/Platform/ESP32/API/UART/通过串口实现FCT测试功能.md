```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "driver/gpio.h"
#include "driver/timer.h"
#include "esp_timer.h"
#include "driver/uart.h"
 
#define TXD1_PIN (GPIO_NUM_17) //串口1的发送数据引脚
#define RXD1_PIN (GPIO_NUM_16) //串口1的接收数据引脚
#define BUF_SIZE (1024) //接收数据缓存大小,该大小需要大于内部FIFO大小:UART_FIFO_LEN(128)
 
#define BUF_SEND_SIZE (1024) //发送数据缓存大小,该大小需要大于内部FIFO大小:UART_FIFO_LEN(128)
 
 
static QueueHandle_t QueueHandle_t_uart1;
 
/*串口任务*/
static void uart_task(void *arg)
{
    /*配置串口参数*/
    uart_config_t uart_config = {
        .baud_rate = 115200,//波特率
        .data_bits = UART_DATA_8_BITS,//数据位8位
        .parity    = UART_PARITY_DISABLE,//无奇偶校验
        .stop_bits = UART_STOP_BITS_1,//停止位1位
        .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,//不使用硬件流控
        .source_clk = UART_SCLK_APB,//串口使用的时钟
    };
    /*初始化串口1*/
    uart_driver_install(UART_NUM_1, 
        BUF_SIZE, //串口1接收缓存大小
        BUF_SEND_SIZE, //串口1发送缓存大小
        10, //队列大小为10
        &QueueHandle_t_uart1, //缓存管理
        0 //设置串口中断优先级,设置为0意味着让系统从1-3级中自动选择一个
    );
    /*设置串口参数*/
    uart_param_config(UART_NUM_1, &uart_config);
    /*设置串口的TX,RX,RTS,DTR引脚*/             //不使用RTS,DTR
    uart_set_pin(UART_NUM_1, TXD1_PIN, RXD1_PIN, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE);
    // 设置串口模式 485半双工通讯模式
    // ESP_ERROR_CHECK(uart_set_mode(uart_num, UART_MODE_RS485_HALF_DUPLEX));
    // 设置UART TOUT功能的读取超时 
    // ESP_ERROR_CHECK(uart_set_rx_timeout(uart_num, ECHO_READ_TOUT));
    /*申请一块内存,用于临时存储接收的数据*/
    uint8_t *data = (uint8_t *) malloc(BUF_SIZE);
    uart_event_t event;
    while (1) {
        if(xQueueReceive(QueueHandle_t_uart1, (void * )&event, portMAX_DELAY))
        {
            switch(event.type) {
                case UART_DATA://接收到数据
                    //读取接收的数据
                    uart_read_bytes(UART_NUM_1, data, event.size, portMAX_DELAY);
                    Uart_DataAnalysis(dtmp,event.size);
                    //返回接收的数据
                    uart_write_bytes(UART_NUM_1, (const char*) data, event.size);
                    break;
                case UART_FIFO_OVF://FIFO溢出(建议加上数据流控制)
                    uart_flush_input(UART_NUM_1);
                    xQueueReset(QueueHandle_t_uart1);
                    break;
                case UART_BUFFER_FULL://接收缓存满(建议加大缓存 BUF_SIZE)
                    uart_flush_input(UART_NUM_1);
                    xQueueReset(QueueHandle_t_uart1);
                    break;
                case UART_BREAK://检测到接收数据中断
                    break;
                case UART_PARITY_ERR://数据校验错误
                    break;
                case UART_FRAME_ERR://数据帧错误
                    break;
                case UART_PATTERN_DET://接收到相匹配的字符(没用到)
                    break;
                default:
                    break;
            }
        }
    }
    free(data);
    data = NULL;
    vTaskDelete(NULL);
}
//串口事件信息分析处理
void Uart_DataAnalysis(uint8_t *str, int size)
{
    char* rest = NULL;
    int mark = 0;
    char* value = NULL;
    char scmd [2] = {0};
    char delim = ':';
    char* stemp = (char*)str;
 
    printf("read usrt:%s\n", stemp);
 
    rest  = strncpy(scmd,stemp, 1); //获取命令
    if(rest == NULL)
    {
        printf("uart input scmd format is error.\n");
        return;
    }
    mark  = strfindsub(stemp,delim);
    value = mysubstr(stemp,mark+1,(size - mark-1)); //获取命令携带的信息
    int cmd = atoi(scmd);
    switch (cmd)
    {
        case FCT_CMD_WRITESN_FCT:         // 写入SN
            printf("FCT write sn read value:%s\n", (char *)value);
            break;
        case FCT_CMD_REQUEST_FCT: // 请求进入FCT
            printf("FCT start read value:%s\n", (char *)value);
            break;
        case FCT_CMD_SET_NET_STAR: // 开始配网
 
            break;
        case FCT_CMD_SET_NET_STEP: // 断开网络
 
            break;
        case FCT_CMD_SET_WIFI_INFO: // 获取wifi信息
 
            break;
        default:
            break;
    }
}
//获取字符串中固定字符第一次出现的位置
int strfindsub(const char *haystack, char ch)
{
    int cnt = 0;
    while (haystack[cnt] != ch)
    {
            cnt++;
    }
 
    return cnt;
}
//拷贝字符串指定位置间的子字符串
char* mysubstr(char* srcstr, int offset, int length) 
{
    assert(length > 0);
    assert(srcstr != NULL);
 
    int total_length = strlen(srcstr);//首先获取srcstr的长度
    //判断srcstr的长度减去需要截取的substr开始位置之后，剩下的长度
    //是否大于指定的长度length，如果大于，就可以取长度为length的子串
    //否则就把从开始位置剩下的字符串全部返回。
    int real_length = ((total_length - offset) >= length ? length : (total_length - offset)) + 1;
    char *tmp;
    if (NULL == (tmp=(char*) malloc(real_length * sizeof(char)))) {
        printf("Memory overflow . \n");
        exit(0);
    }
    strncpy(tmp, srcstr+offset, real_length - 1);
    tmp[real_length - 1] = '\0';
 
    return tmp;
}
void app_main(void)
{
    xTaskCreate(uart_task, "uart_task", 2048, NULL, 10, NULL);
}
```

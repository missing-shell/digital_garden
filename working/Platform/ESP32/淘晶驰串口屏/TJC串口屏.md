* 型号：TJC4848X540_011CS_I_Y
```c
 //以下代码只在上电时运行一次,一般用于全局变量定义和上电初始化数据
int sys0=0,sys1=0,sys2=0     //全局变量定义目前仅支持4字节有符号整形(int),不支持其他类型的全局变量声明,如需使用字符串类型可以在页面中使用变量控件来实现
baud=9600//配置波特率
dim=100//配置屏幕背光
recmod=0//串口协议模式:0-字符串指令协议;1-主动解析协议
printh 00 00 00 ff ff ff 88 ff ff ff//输出上电信息到串口
page 0                       //上电刷新第0页

```
* 基本通信实例：[[../API/UART/V1.1_uart_echo]]
### 通信格式
* 帧头 + 帧长度 + 帧内容 + 帧校验 + 帧尾
* 目前淘晶驰串口屏仅支持8N1通讯格式，即8位数据位，无校验位，1位停止位
* 不支持5/6/7位数据位，不支持奇校验和偶校验，也不支持1.5/2位停止位
#### 单片机和串口屏的通信
* 正确的接线方法是：单片机的RX接屏的TX，单片机的TX接屏的RX。
* 单片机的通信波特率要和屏的一致，串口屏的默认波特率是9600，数据格式8-0-1（1位起始位，8位数据位，无校验位，1位结束位）。
* 单片机控制屏的指令格式，一条控制指令，一条结束符，控制指令见指令集的指令，结束符是16进制的3个FF(FF FF FF).
* 可以用串口助手监视单片机发过来的数据。
* **根据自己的需求选择合适的编码，这里的编码要和串口屏的编码一致。**
* **
* 发送指令：单片机串口通过字符串模式发送t0.txt="adc";
* 发送结束符：单片机通过HEX模式发送0xff,0xff,0xff;
* **
### 数据发送
### `bands`-上电默认波特率值
* 通常配置在program.s文件中，请写在page指令的前面，page指令后面的代码是不会执行的

#### 字符串
* 控件为`t0.txt="%s"`
##### 发送字符串`"abcded"`
```c++
	char str_send[] = "t0.txt=\"abcded\"\xff\xff\xff";
	uart_write_bytes(ECHO_UART_PORT_NUM, str_send, strlen(str_send));
	ESP_LOGI(TAG, "Send str: %s", (char *)str_send); // Log the received data
```
##### 发送字符串`s`
```c++
	char *s="hello";
	char str_send[100];//不能太大，以免栈溢出
    snprintf(str_send, sizeof(str_send), "t0.txt=\"%s\"\xff\xff\xff", s);
    uart_write_bytes(ECHO_UART_PORT_NUM, str_send, strlen(str_send));
```
##### 字符串拼接
```c++
int gz_buf=12345;
    char cgq_buf[BUF_SIZE];
    memset(cgq_buf,0,sizeof(cgq_buf));
    strcpy(cgq_buf,"t");
    strcat(cgq_buf,"0");
    strcat(cgq_buf,".txt=\"");
    strcat(cgq_buf,"\x41\x42\x43");
    strcat(cgq_buf,"\0");
    strcat(cgq_buf,"\"");
    uart_write_bytes(ECHO_UART_PORT_NUM, cgq_buf, strlen(cgq_buf));
    uart_write_bytes(ECHO_UART_PORT_NUM, "\xff\xff\xff", strlen("\xff\xff\xff"));
```
#### 数字
* 控件为`n0.val=%d`
##### 发送数字`256`
```c++
	char val_send[] = "n0.val=256\xff\xff\xff";
	uart_write_bytes(ECHO_UART_PORT_NUM, val_send, strlen(val_send));
```
##### 发送数字`num`
```c++
	int num = 12222;
    char val_send[100];//不能太大，以免栈溢出
    sprintf(val_send,"n0.val=%d\xff\xff\xff", num);
    uart_write_bytes(ECHO_UART_PORT_NUM, val_send, strlen(val_send));
```
#### 虚拟浮点数
* 虚拟浮点数控件赋值必须是整形（int），否则会出错,虚拟浮点数本质上还是整数
* 并且需要提前规定控件的整数和小数位的长度
* 控件为`x0.val=%d\xff\xff\xff",a`
##### 可以使用字符串模式替代
```c++
	float p=3.124;
	char str_send[100];//不能太大，以免栈溢出
    snprintf(str_send, sizeof(str_send), "t0.txt=\"%.3f\"\xff\xff\xff",p);
    uart_write_bytes(ECHO_UART_PORT_NUM, str_send, strlen(str_send));
```
#### tips
* 命令采用字符串类型发送
* 结尾符号就是连续发送三个字节（8bit）0XFF;
* 建议用户MCU完成初始化以后延时一段时间再发数据给串口屏。对于T0和K0系列我们建议延时250MS,对于X3和X5系列建议延时1.5S。

### 串口调试连接规则
* 任一时间,只允许一对一连接,有以下3种情况:
* [模拟器(电脑)与串口屏实物连接](http://wiki.tjc1688.com/debug/base/simulator2tjc.html)
* [模拟器(电脑)与单片机连接](http://wiki.tjc1688.com/debug/base/simulator2mcu.html)
* [串口屏与单片机连接](http://wiki.tjc1688.com/debug/base/tjc2mcu.html)

## 屏幕
串口袋实验室屏幕方向为：竖屏，0度
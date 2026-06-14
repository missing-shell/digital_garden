* 捕获模拟电信号（例如麦克风捕获的声音）并将其转换为一系列数字“数字”值以由数字计算机或 DSP 存储/处理的过程。用于此转换过程的电子设备已知为 A/D 或**ADC**（模数转换器）。
* 可以通过降低模拟电压参考来提高 ADC 转换分辨率（前提是您正在测量小信号）。
* 最小分辨率为 9 位，此时 ADC 转换范围为 (0 – 511)。最大(0~4095)

##### 使用ESP32读取到模拟量电压值的步骤是：
* 取样
* 获取特征曲线（只需一次获取，多次使用）
* 调用 API 根据特征曲线转换测量的 Voltage 值

##### **采样率**
* 采集频率 = ADC采样率 / ADC采样时间
* ADC 将连续模拟信号转换为数字数据的速率称为“采样率”。如果转换单个样本需要 Ts 时间，则该 ADC 的采样率为 Fs = 1/Ts。然后可以通过数学插值从离散时间数字值再现原始模拟信号。此过程的精度由采样率和量化误差的综合影响决定。

##### **抽样定理**

理论上，为了获得有关原始模拟信号的最少信息，ADC 必须以 Fs >= 2F MAX的频率对模拟信号进行采样和转换，这满足香农-奈奎斯特采样定理。

**Fs** -> ADC 的采样频率

**F MAX** -> 正在转换的模拟信号的最大频率

##### init_config .ADC_ULP_MODE_DISABLE

`ADC_ULP_MODE_DISABLE`是一个宏定义，用于配置ESP32-C6的ADC在Ultra Low Power（ULP，超低功耗）模式下的工作状态。当设置为`ADC_ULP_MODE_DISABLE`时，表示禁用ULP模式[forum.arduino.cc](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s2/api-reference/peripherals/adc.html)。

在ESP32-C6的ADC配置中，`ulp_mode`字段决定ADC是否运行在ULP模式。这个模式是ESP32-C6的一种特殊的低功耗模式，可以在系统的主CPU关闭或者进入深度睡眠模式的时候，继续运行ADC以读取模拟信号[docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/sleep_modes.html)。

在此模式下，ADC可以被配置为定期自动读取模拟信号，并将读取的数据存储在一个特定的内存区域。当主CPU重新启动或唤醒后，可以直接从这个内存区域读取数据，而无需等待ADC的读取过程。这样可以在一定程度上提高系统的响应速度和效率[docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/sleep_modes.html)。


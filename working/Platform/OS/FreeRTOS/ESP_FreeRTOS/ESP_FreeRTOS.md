* 由于 ESP32-C6 是单核 SoC，因此始终设置[CONFIG_FREERTOS_UNICORE配置。](https://docs.espressif.com/projects/esp-idf/zh_CN/v5.1.1/esp32c6/api-reference/kconfig.html#config-freertos-unicore)
* ESP-IDF 将自动启动 FreeRTOS。用户必须定义​​一个函数作为用户应用程序的入口点，并在 ESP-IDF 启动时自动调用。`void app_main(void)`
* FreeRTOS 端口确保 FreeRTOS 分配的所有动态内存都放置在内部存储器中。
* CPU0 和 CPU1 的“PRO_CPU”和“APP_CPU”别名存在于 ESP-IDF 中，因为它们反映了典型的 IDF 应用程序将如何利用这两个 CPU。
* 通常，负责处理无线网络（例如 WiFi 或蓝牙）的任务将固定到 CPU0（因此名称为 PRO_CPU），而处理应用程序其余部分的任务将固定到 CPU1（因此名称为 APP_CPU）。
* 在 FreeRTOS 中，定时器和主任务是**并行**运行的。定时器的回调函数 `TIME_timer_callback` 是在定时器服务/守护任务中执行的，这个任务有自己的优先级和堆栈。因此，即使主任务正在延迟或执行其他操作，定时器仍然会按照预定的时间触发。
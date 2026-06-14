### 基本中断方式
```c++
 void IRAM_ATTR gpio_isr_handler(void* arg)
{
    g_pin_triggered = (int) arg;  // 把触发中断的 GPIO 编号存到全局变量中

    // 在这里，你可以添加其他处理中断的代码。但是，请记住，中断处理程序应尽可能快地执行完毕，以防止阻塞其他的中断。
    // 因此，你应该避免在这里进行复杂的操作，比如打印信息、与硬件通讯，或执行耗时的计算。一般来说，一个好的实践是
    // 将需要在任务上下文中执行的操作存到全局变量中，然后在任务中检查这个全局变量。
}
/*这里的 `gpio_isr_handler` 是一个中断服务程序（ISR）处理器。当中断事件发生时，这个函数会被调用。这个函数获取产生中断的 GPIO编号，然后将它发送到一个由 FreeRTOS 管理的队列中。请注意 `IRAM_ATTR` 属性，意味着这个函数放在内部RAM中，这对于中断处理非常重要，因为它可以避免潜在的 Flash 访问问题。*/
```

### 标准中断方式
##### 1.定义并实现中断服务程序
```c++
static void IRAM_ATTR gpio_isr_handler(void* arg)
{
    uint32_t gpio_num = (uint32_t) arg;
    xQueueSendFromISR(gpio_evt_queue, &gpio_num, NULL);
}
```

##### 2. 配置 GPIO 中断：
```c++
io_conf.intr_type = GPIO_INTR_POSEDGE;  // 设置为上升沿触发的中断
io_conf.pin_bit_mask = GPIO_INPUT_PIN_SEL;  //设置需要设置的GPIO
io_conf.mode = GPIO_MODE_INPUT;  //设置为输入模式
io_conf.pull_up_en = 1;  //开启上拉模式
gpio_config(&io_conf); 

gpio_set_intr_type(GPIO_INPUT_IO_0, GPIO_INTR_ANYEDGE);  // 设置GPIO的中断类型，此处设置为任何边沿触发中断

//初始化安装GPIO中断服务
gpio_install_isr_service(ESP_INTR_FLAG_DEFAULT);

//为特定的GPIO引脚添加中断服务程序处理器
gpio_isr_handler_add(GPIO_INPUT_IO_0, gpio_isr_handler, (void*) GPIO_INPUT_IO_0);
gpio_isr_handler_add(GPIO_INPUT_IO_1, gpio_isr_handler, (void*) GPIO_INPUT_IO_1);
```

##### 3. 从队列接收并处理中断事件
```c++
static void gpio_task_example(void* arg)
{
    uint32_t io_num;
    for(;;) {
        if(xQueueReceive(gpio_evt_queue, &io_num, portMAX_DELAY)) {
            printf("GPIO[%"PRIu32"] intr, val: %d\n", io_num, gpio_get_level(io_num));
        }
    }
}
/*此 FreeRTOS 任务会无限循环地检查由 ISR 处理器发送到队列的消息。当它从队列接收到一个消息（使用`xQueueReceive`函数），它会打印出产生中断的 GPIO 号以及它的当前状态。*/
```

##### 所有这些步骤合起来，实现了一个基于事件驱动的编程模型，当一个 GPIO 中断事件发生时，相关的处理程序会被自动调用处理这个事件。


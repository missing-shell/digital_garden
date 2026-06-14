### kernel-features
> 针对单读取器单写入器场景进行了优化
- 例如将数据从中断服务例程传递到任务，或者在**双核 CPU** 上从一个微控制器核心传递到另一个微控制器核心。
- 数据通过**复制传递** - 发送方将数据复制到缓冲区中，读取方将数据从缓冲区中复制出来。

> 流缓冲区实现使用 [direct to task notifications](https://www.freertos.org/Documentation/02-Kernel/02-Kernel-features/03-Direct-to-task-notifications/01-Task-notifications)。因此，调用将调用任务置于阻塞状态的流缓冲区 API 函数可以更改调用任务的**通知状态**和**值**。
### 适用场景
> 处理连续数据流的场景
> 适合传输如文件、网络数据包或长消息等**不定长**数据。

### 与message buffer的区别
流缓冲区传递*连续*的字节流。消息缓冲区传递大小可变但离散的消息。
消息缓冲区使用流缓冲区进行数据传输。
## 底层实现
- *数据结构*: 流缓冲区在内部维护一个==环形缓冲区==（也称为循环缓冲区或环形队列）避免缓冲区满或空时的复杂边界条件处理。包含两部分：用于存储数据的实际数据区域和两个指针（读指针和写指针）
- *锁定机制*: 为了保证在多任务环境下的==线程安全==，流缓冲区在读写操作时通常需要某种形式的锁定机制，尽管FreeRTOS的流缓冲区设计倾向于减少锁定需求，特别是在单生产者单消费者场景中。这可能通过原子操作或轻量级的互斥锁（如果可用）来实现，以最小化调度开销。
- *数据传输*: 数据不是直接在任务或中断上下文间共享，而是通过==拷贝==方式从生产者复制到缓冲区，再从缓冲区复制到消费者。简化了同步问题，允许生产者和消费者独立工作，不必同时活动。
- *可变长度数据*: 与*固定大小*的数据包通信（如消息队列）不同，流缓冲区允许写入和读取==任意长度==的数据，只要不超过缓冲区的最大容量。
- *优化策略*: 实现中可能包含一些优化策略，比如**批量读写**、避免不必要的内存拷贝、以及对齐和缓存管理等，以提高数据传输效率。
- **中断安全**: FreeRTOS确保流缓冲区的操作可以安全地在中断服务程序（`ISR`）中使用，这意味着可以在中断上下文中向缓冲区写入数据或从中读取，而不会干扰到其他任务。
- **API接口**: FreeRTOS提供了创建、发送、接收数据到流缓冲区的API，如`xStreamBufferCreate`用于创建流缓冲区，`xStreamBufferSend`和`xStreamBufferReceive`分别用于发送和接收数据。

### StreamBuffer_t
```c
/* Structure that hold state information on the buffer. */
typedef struct StreamBufferDef_t
{
    volatile size_t xTail;                       /* Index to the next item to read within the buffer. */
    volatile size_t xHead;                       /* Index to the next item to write within the buffer. */
    size_t xLength;                              /* The length of the buffer pointed to by pucBuffer. */
    size_t xTriggerLevelBytes;                   /* The number of bytes that must be in the stream buffer before a task that is waiting for data is unblocked. */
    volatile TaskHandle_t xTaskWaitingToReceive; /* Holds the handle of a task waiting for data, or NULL if no tasks are waiting. */
    volatile TaskHandle_t xTaskWaitingToSend;    /* Holds the handle of a task waiting to send data to a message buffer that is full. */
    uint8_t * pucBuffer;                         /* Points to the buffer itself - that is - the RAM that stores the data passed through the buffer. */
    uint8_t ucFlags;

    #if ( configUSE_TRACE_FACILITY == 1 )
        UBaseType_t uxStreamBufferNumber; /* Used for tracing purposes. */
    #endif

    #if ( configUSE_SB_COMPLETED_CALLBACK == 1 )
        StreamBufferCallbackFunction_t pxSendCompletedCallback;    /* Optional callback called on send complete. sbSEND_COMPLETED is called if this is NULL. */
        StreamBufferCallbackFunction_t pxReceiveCompletedCallback; /* Optional callback called on receive complete.  sbRECEIVE_COMPLETED is called if this is NULL. */
    #endif
    UBaseType_t uxNotificationIndex;                               /* The index we are using for notification, by default tskDEFAULT_INDEX_TO_NOTIFY. */
} StreamBuffer_t;
```
## API
### xStreamBufferCreate, xStreamBufferCreateWithCallback
Stream buffers created using the `xStreamBufferCreate()` API share the *same* `send and receive completed callback` functions, which are defined using the `sbSEND_COMPLETED()` and `sbRECEIVE_COMPLETED()` macros.
Stream buffers created using the `xStreamBufferCreateWithCallback()` API can each have their own *unique* send and receive completed callback functions.
#### xTriggerLevelBytes
- If a reading task's `block time` expires before the `trigger level` is reached then the task will still **receive** as many bytes as are actually available.
- Setting a trigger level of `0` will *result* in a trigger level of `1` being used.
- It is *not valid* to specify a trigger level that is `greater than` the buffer size.
#### pxSendCompletedCallback
If the parameter is `NULL`, the default implementation provided by the `sbSEND_COMPLETED` macro is used.

The prototype defined by StreamBufferCallbackFunction_t, which is:
```c
void vSendCallbackFunction( StreamBufferHandle_t xStreamBuffer,  
                            BaseType_t xIsInsideISR,  
                            BaseType_t * const pxHigherPriorityTaskWoken );  
```
#### sbSEND_COMPLETED()
`sbSEND_COMPLETED() `是一个宏，当数据写入使用 `xStreamBufferCreate()` 或 `xStreamBufferCreateStatic(`) API 创建的流缓冲区时，会调用该宏（在 `FreeRTOS` API 函数内部）。它采用单个参数，即已更新的流缓冲区的*句柄*。

默认情况下（如果应用程序编写者不提供他们自己的**宏实现**），sbSEND_COMPLETED() 会检查流缓冲区上是否有任务被阻止以等待数据，如果是，则将该任务从**阻止状态**中*移除*。

```c
void vSendCallbackFunction( StreamBufferHandle_t xStreamBuffer,  
                            BaseType_t xIsInsideISR,  
                            BaseType_t * const pxHigherPriorityTaskWoken )  
{  
    /* Insert code here which is invoked when a data write operation  
     * to the stream buffer causes the number of bytes in the buffer  
     * to be more then the trigger level.  

     * This is useful when a stream buffer is used to pass data between  
     * cores on a multicore processor. In that scenario, this callback  
     * can be implemented to generate an interrupt in the other CPU core,  
     * and the interrupt's service routine can then use the  
     * xStreamBufferSendCompletedFromISR() API function to check, and if  
     * necessary unblock, a task that was waiting for the data. */  
}  

void vReceiveCallbackFunction( StreamBufferHandle_t xStreamBuffer,  
                               BaseType_t xIsInsideISR,  
                               BaseType_t * const pxHigherPriorityTaskWoken )  
{  
    /* Insert code here which is invoked when data is read from a stream  
     * buffer.  

     * This is useful when a stream buffer is used to pass data between  
     * cores on a multicore processor. In that scenario, this callback  
     * can be implemented to generate an interrupt in the other CPU core,  
     * and the interrupt's service routine can then use the  
     * xStreamBufferReceiveCompletedFromISR() API function to check, and if  
     * necessary unblock, a task that was waiting to send the data. */  
}  

void vAFunction( void )  
{  
StreamBufferHandle_t xStreamBuffer, xStreamBufferWithCallback;  
const size_t xStreamBufferSizeBytes = 100, xTriggerLevel = 10;  
  
    /* Create a stream buffer that can hold 100 bytes and uses the  
     * functions defined using the sbSEND_COMPLETED() and  
     * sbRECEIVE_COMPLETED() macros as send and receive completed  
     * callback functions. The memory used to hold both the stream  
     * buffer structure and the data in the stream buffer is  
     * allocated dynamically. */  
    xStreamBuffer = xStreamBufferCreate( xStreamBufferSizeBytes,  
                                         xTriggerLevel );  

    if( xStreamBuffer == NULL )  
    {  
        /* There was not enough heap memory space available to create the  
           stream buffer. */  
    }  
    else  
    {  
        /* The stream buffer was created successfully and can now be used. */  
    }  
      
    /* Create a stream buffer that can hold 100 bytes and uses the  
     * functions vSendCallbackFunction and vReceiveCallbackFunction  
     * as send and receive completed callback functions. The memory  
     * used to hold both the stream buffer structure and the data  
     * in the stream buffer is allocated dynamically. */  
    xStreamBufferWithCallback = xStreamBufferCreateWithCallback(   
                                    xStreamBufferSizeBytes,  
                                    xTriggerLevel,  
                                    vSendCallbackFunction,  
                                    vReceiveCallbackFunction );  

    if( xStreamBufferWithCallback == NULL )  
    {  
        /* There was not enough heap memory space available to create the  
         * stream buffer. */  
    }  
    else  
    {  
        /* The stream buffer was created successfully and can now be used. */  
    }  
}  
```
### uxStreamBufferGetStreamBufferNotificationIndex（）
Retrieves the task notification index used for the supplied [stream buffer](https://www.freertos.org/Documentation/02-Kernel/04-API-references/08-Stream-buffers/00-RTOS-stream-buffer-API), which can be set using [vStreamBufferSetStreamBufferNotificationIndex](https://www.freertos.org/Documentation/02-Kernel/04-API-references/08-Stream-buffers/17-vStreamBufferSetStreamBufferNotificationIndex). If the task notification index for the stream buffer is not changed using `vStreamBufferSetStreamBufferNotificationIndex`, this function returns the default value `tskDEFAULT_INDEX_TO_NOTIFY`.

```c
UBaseType_t uxStreamBufferGetStreamBufferNotificationIndex( StreamBufferHandle_t xStreamBuffer );
```
*Parameters:*
- `xStreamBuffer` The handle of the stream buffer for which the task notification index is retrieved.
*Returns:*
- The task notification index used for the stream buffer.
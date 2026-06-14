## 概念
- FreeRTOS-硬实时操作系统
- 可以为实现硬实时要求的线程分配更高的优先级，并为实现软实时要求的线程分配更低的优先级。 这将确保硬实时线程始终在软实时线程之前执行
- 在 FreeRTOS 中，每个执行线程都称为一个 “任务”。
- FreeRTOS 源代码代码显式限定 `char` 的每个用户都使用 `signed` 或 `unsigned`，除非 char 用于保存 ASCII 字符，或者指向 `char` 的指针用于指向字符串。
- 内核对象，如任务，队列，信号量和事件组。 为了使 FreeRTOS 尽可能易于使用，这些内核对象在编译时不是分配的，而是在**运行时动态分配**的; 每次创建内核对象时，FreeRTOS 都会分配 RAM，并在每次删除内核对象时释放 RAM。
### 变量名
变量的前缀是它们的类型：

- `c` 表示 `char`
- `s` 表示 `int16_t`（`short`）
- `l`表示 `int32_t`（`long`）
- `x` 表示 `BaseType_t` 和任何其他非标准类型（结构，任务句柄，队列句柄等）

如果变量是无符号的，则它也带有 `u` 前缀。 如果变量是指针，则它也带有 `p` 前缀。 例如，`uint8_t` 类型的变量将以 `uc` 为前缀，而指向 `char` 的类型指针的变量将以 `pc` 为前缀。

### 函数名
函数以它们返回的类型和它们在其中定义的文件为前缀。 例如：

- _v_**Task**PrioritySet() 返回一个 `void`，并定义在 **task**.c 中。
- _x_**Queue**Receive() 返回一个 `BaseType_t`类型的变量，并定义在 **queue**.c 中。
- _pv_**Timer**GetTimerID() 返回一个 `void`类型的指针，并定义在 **timers**.c 中。

文件范围（私有）函数以 `prv` 为前缀。

## 全局变量
- 全局变量的类型必须等于CPU和内存的通道

### Race Condition
- **竞争冒险**
- 使用信号量保护共享资源，临界资源
### mutex
1. 创建锁
2. 获取钥匙
3. 释放锁
## 分配核心
- 为任务指定内核0
```c
xTaskCreatePinnedToCore(core_task, "core_task", 1024 * 4, NULL, 1, NULL, 0);
```
### 查看所有任务
```c
void task_list(void)
{
    char ptrTaskList[250];
    vTaskList(ptrTaskList);
    printf("*******************************************\n");
    printf("Task            State   Prio    Stack    Num\n");
    printf("*******************************************\n");
    printf(ptrTaskList);
    printf("*******************************************\n");
}
```
## 定时器
---
- `vTaskDelayUntil` 和 `timer `都可以用来实现间隔固定时间执行任务，但是它们有一些不同之处。
- `vTaskDelayUntil` 是一个 `FreeRTOS` 函数，它用于将任务延迟到固定的时间间隔执行。它的工作原理是，调用这个函数的任务会被挂起直到指定的时间间隔到达，然后再恢复执行。这个函数的优点是可以**保证任务间隔固定的执行时间**，使得任务的执行更加精确。
- 定时器是一种常见的系统资源，它可以在指定的时间间隔到达时触发某些事件。在 FreeRTOS 中，也可以使用定时器来实现间隔固定时间执行任务。定时器的**优点是它可以在不同的任务之间共享**，可以支持多个定时器同时工作。
- 对于两者的时间精度，在 FreeRTOS 中，定时器的时间精度更高。在 FreeRTOS 中，定时器的时间精度是由系统的调度周期决定的，通常情况下是 10ms。而 vTaskDelayUntil 的时间精度受到任务调度的影响，因此可能不如定时器的时间精度高。
- 如果你需要任务间隔固定的执行时间，并且希望任务之间的执行关系清晰明了（**固定顺序**），你可以考虑使用 vTaskDelayUntil。
- 如果你需要一个可以在不同任务之间共享的定时器，或者你希望更高的时间精度，你可以考虑使用定时器（**执行顺序没有要求**）。
---
### `vTaskDelayUntil `
### 软件定时器
#### 单次定时
#### 多次定时

## 内存优化
### 系统调用时栈的变化
- **触发 `SVC` 异常**：自动保存一些关键寄存器的信息到栈中，栈指针向下移动。
- **进入异常处理程序**：从栈中读取服务编号，执行相应的系统调用处理逻辑。
- **返回用户模式**：恢复之前保存的寄存器内容，栈指针调整回原来的位置。
#### 1. 触发 `SVC` 异常
当线程执行 `SVC` 指令时，会触发一个 `SVC` 异常。异常处理机制会自动保存一些关键寄存器的信息到栈中。

> 自动保存的寄存器

- **PC（程序计数器）**：保存当前指令的地址加2（因为 `SVC` 指令本身占用2个字节）。
- **LR（链接寄存器）**：保存返回地址，即 `SVC` 指令后的下一条指令地址。
- **PSR（程序状态寄存器）**：保存当前处理器状态，包括标志位等。
- **通用寄存器**：根据具体实现，可能会保存一些通用寄存器的内容。

> 栈指针调整

- **栈指针**：栈指针（SP）会被调整，以容纳上述保存的数据。具体来说，栈指针会向下移动（减小），为这些数据分配空间。
#### 2. 进入异常处理程序
异常处理程序（如 `SVC_Handler`）会被调用，处理系统调用请求。

> 获取服务编号

- **服务编号**：在 `SVC` 指令中，通常会传递一个服务编号，用于区分不同的系统调用。异常处理程序需要从栈中读取这个服务编号。
  ```c
  uint32_t svcNumber = __get_PSP();  // 假设使用 PSP 作为栈指针
  svcNumber = *(uint32_t *)(svcNumber + 8);  // 从栈中读取服务编号
  ```

> 执行系统调用

- **系统调用处理**：根据服务编号，执行相应的系统调用处理逻辑。
  ```c
  switch (svcNumber) {
      case 0:
          // 处理服务编号为 0 的系统调用
          vTaskCreate(taskFunction, "Task", 1024, NULL, 1, NULL);
          break;
      case 1:
          // 处理服务编号为 1 的系统调用
          vTaskDelete(NULL);
          break;
      // 其他服务编号
      default:
          // 未知服务编号
          break;
  }
  ```
#### 3. 返回用户模式
系统调用处理完成后，需要恢复线程的状态，返回到用户模式继续执行。

> 恢复寄存器

- **恢复寄存器**：将之前保存的寄存器内容从栈中恢复。
  ```c
  asm volatile (
      "ldmia sp!, {r0-r12, lr, pc}^ \n"  // 恢复寄存器
  );
  ```

> 调整栈指针

- **栈指针调整**：栈指针会被调整回原来的位置，释放之前分配的空间。
  ```c
  sp = original_sp;  // 恢复原来的栈指针
  ```
### 栈大小不够了，如何扩充
1. _静态增加栈大小_：在任务创建时指定更大的栈大小。
2. _动态增加栈大小_：手动分配额外栈空间并调整栈指针。
3. _使用堆栈溢出钩子_：检测栈溢出并采取相应措施。
4. _使用更大的默认栈大小_：在 `FreeRTOSConfig.h` 中设置更大的默认栈大小。
5. _优化代码减少栈使用_：通过代码优化减少栈的使用量。
#### 静态增加栈大小
在创建任务时，可以_静态_地指定更大的栈大小。这是最直接的方法，适用于任务创建时已知所需栈大小的情况。
#### 使用堆栈溢出钩子
`FreeRTOS`提供了一个堆栈溢出钩子函数 `vApplicationStackOverflowHook`，可以在栈溢出时调用该函数进行处理。可以用来检测栈溢出并采取相应措施。
```c
void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    // 栈溢出处理
    configASSERT(xTask);
    (*void*)pcTaskName;
    for (;;) {
        // 无限循环，防止程序继续运行
    }
}
```
#### 用更大的默认栈大小

在 `FreeRTOSConfig.h` 文件中，可以设置默认的栈大小。这样，所有任务都会使用更大的栈空间。
```c
#define configMINIMAL_STACK_SIZE 2048
```

### `Memory Allocation`
- 大多数微控制器系统中的易失性存储器（例如 RAM）分为 3 个部分：静态、堆栈和堆。
- 静态内存用于存储全局变量和在代码中指定为“静态”的变量（它们在函数调用之间持续存在）。 
- 堆栈用于局部变量的自动分配。堆栈内存被组织为后进先出（LIFO）系统，以便在调用新函数时可以将一个函数的变量“推入”堆栈。返回到第一个函数后，该函数的变量可以“弹出”，函数可以使用这些变量从中断处继续运行。
- 堆必须由程序员显式分配。在 C 语言中，您最常使用 malloc() 函数为变量、缓冲区等分配堆。这称为“动态分配”。请注意，在没有垃圾收集系统的语言中（例如 C 和 C++ ），您必须在不再使用堆内存时将其释放。如果不这样做将导致内存泄漏，并可能导致未定义的影响，例如损坏内存的其他部分。
![[Memory Allocation.jpg]]

### `RTOS Memory Allocation`
- 当您在FreeRTOS 中创建任务时（例如使用xTaskCreate()），操作系统将为该任务分配一段堆内存。
![[RTOS Memory Allocation.jpg]]
### Stack
- 分配的内存的一部分是任务控制块（`TCB`），它用于存储有关任务的信息，例如其优先级和本地堆栈指针。另一部分保留为本地堆栈，其运行方式与全局堆栈类似（但仅针对该任务规模较小）。
### Heap
- 使用` FreeRTOS `时，`malloc()` 和` free() `不被认为是线程安全的。因此，建议您改用` pvPortMalloc() `和 `vPortFree()`。使用这些时，将从系统的全局堆（而不是为任务分配的堆）分配内存。
### Static
- `FreeRTOS` 的最新版本允许创建[静态任务](https://www.freertos.org/xTaskCreateStatic.html)。它们仅使用静态内存（在静态内存而不是堆中分配自己的本地堆栈和 `TCB`）。这对于您不能或不想使用堆内存来防止堆溢出的情况很有用。
### 内存优化
- 推荐为任务分配器内存大小的两倍的内存
- 不推荐在任务中使用`printf`，会消耗内存
## 任务管理
### PendSV和SVC
- **`PendSV` 异常**：主要用于任务调度和上下文切换。当调度器需要切换当前任务时，会设置 `PendSV` 异常，异常处理程序会保存当前任务的上下文信息，并恢复新任务的上下文信息。
- **`SVC` 异常**：主要用于系统调用和服务请求。应用程序可以通过 `SVC` 指令触发异常，传递一个**服务编号**，异常处理程序会根据服务编号执行相应的系统调用。
#### PendSV 异常
`PendSV` 异常主要用于任务调度和上下文切换。当调度器需要切换当前任务时，会触发 `PendSV` 异常，从而执行上下文切换操作。

> 功能

- **任务调度**：当调度器检测到有更高优先级的任务就绪时，会设置 `PendSV` 异常，以便在合适的时机进行任务切换。
- **上下文切换**：在 `PendSV` 异常处理程序中，保存当前任务的上下文信息，并恢复新任务的上下文信息。

> 实现

1. **设置 `PendSV` 异常**：
   - 当调度器需要进行任务切换时，会调用 `portYIELD_FROM_ISR()` 或 `vTaskSwitchContext()`，这些函数会设置 `PendSV` 异常。
   ```c
   void vTaskSwitchContext(void) {
       // 设置 PendSV 异常
       SCB->ICSR |= SCB_ICSR_PENDSVSET_Msk;
   }
   ```

2. **处理 `PendSV` 异常**：
   - `PendSV` 异常处理程序会执行上下文切换操作。
   ```c
   void PendSV_Handler(void) {
       // 禁用中断
       __disable_irq();

       // 保存当前任务的上下文
       portSAVE_CONTEXT();

       // 选择下一个任务
       pxCurrentTCB = pxChooseNextTask();

       // 恢复新任务的上下文
       portRESTORE_CONTEXT();

       // 启用中断
       __enable_irq();
   }
   ```

#### SVC 异常
`SVC` 异常主要用于系统调用和服务请求。在FreeRTOS中，`SVC` 异常通常用于实现任务创建、删除、挂起、恢复等操作。

> 功能

- **系统调用**：应用程序可以通过 `SVC` 异常调用内核提供的服务，如任务管理、内存管理等。
- **特权模式切换**：`SVC` 异常可以在用户模式下触发，进入特权模式执行系统调用。

> 实现

1. **触发 `SVC` 异常**：
   - 应用程序通过 `SVC` 指令触发异常，传递一个服务编号（service number）。
   ```c
   __asm(" SVC #0 \n");  // 触发 SVC 异常，服务编号为 0
   ```

2. **处理 `SVC` 异常**：
   - `SVC` 异常处理程序会根据服务编号执行相应的系统调用。
   ```c
   void SVC_Handler(void) {
       uint32_t svcNumber;

       // 获取 SVC 指令中的服务编号
       svcNumber = __get_PSP();  // 假设使用 PSP 作为栈指针
       svcNumber = *(uint32_t *)(svcNumber + 8);  // 从栈中读取服务编号

       switch (svcNumber) {
           case 0:
               // 处理服务编号为 0 的系统调用
               break;
           case 1:
               // 处理服务编号为 1 的系统调用
               break;
           // 其他服务编号
           default:
               // 未知服务编号
               break;
       }
   }
   ```

### FreeRTOS任务切换的过程
在FreeRTOS中，任务切换是一个复杂但有序的过程，涉及上下文保存和恢复、任务状态更新、中断处理和临界区保护等步骤。
RTOS上下文切换#### 1. 上下文切换

上下文切换是任务切换的核心操作，主要包括保存当前任务的上下文信息和恢复新任务的上下文信息。

> 保存当前任务的上下文

1. **禁用中断**：
   - 在开始上下文切换之前，通常需要禁用中断，以防止中断干扰上下文保存过程。
   ```c
   portDISABLE_INTERRUPTS();
   ```

2. **保存寄存器**：
   - 将当前任务的所有寄存器（包括通用寄存器、状态寄存器、程序计数器等）保存到当前任务的堆栈中。
   ```c
   asm volatile (
       "push {r0-r12, lr}\n"  // 保存寄存器
   );
   ```

3. **保存堆栈指针**：
   - 将当前任务的堆栈指针（SP）保存到任务控制块（TCB）中。
   ```c
   pxCurrentTCB->pxStack = sp;  // sp 是当前任务的堆栈指针
   ```

4. **更新任务状态**：
   - 将当前任务的状态从“运行”（Running）更新为“就绪”（Ready）或“阻塞”（Blocked），具体取决于任务为什么被抢占。
   ```c
   pxCurrentTCB->eTaskState = eReady;  // 或 eBlocked
   ```

5. **选择下一个任务**：
   - 调度器选择下一个最高优先级的就绪任务。
   ```c
   pxCurrentTCB = pxChooseNextTask();
   ```

> 恢复新任务的上下文

1. **恢复堆栈指针**：
   - 从新任务的TCB中恢复堆栈指针。
   ```c
   sp = pxCurrentTCB->pxStack;  // sp 是新任务的堆栈指针
   ```

2. **恢复寄存器**：
   - 从新任务的堆栈中恢复所有寄存器的状态。
   ```c
   asm volatile (
       "pop {r0-r12, pc}\n"  // 恢复寄存器
   );
   ```

3. **更新任务状态**：
   - 将新任务的状态从“就绪”（Ready）更新为“运行”（Running）。
   ```c
   pxCurrentTCB->eTaskState = eRunning;
   ```

4. **启用中断**：
   - 在恢复新任务的上下文后，重新启用中断。
   ```c
   portENABLE_INTERRUPTS();
   ```

#### 2. 任务状态更新

在任务切换过程中，需要更新当前任务和新任务的状态。

- **当前任务状态更新**：
  - 将当前任务的状态从“运行”（Running）更新为“就绪”（Ready）或“阻塞”（Blocked）。
  ```c
  pxCurrentTCB->eTaskState = eReady;  // 或 eBlocked
  ```

- **新任务状态更新**：
  - 将新任务的状态从“就绪”（Ready）更新为“运行”（Running）。
  ```c
  pxCurrentTCB->eTaskState = eRunning;
  ```

#### 3. 中断处理

任务切换通常由中断触发，因此中断处理是任务切换的重要部分。

- **中断服务例程（ISR）**：
  - **中断处理**：ISR处理中断事件，例如定时器中断、外部中断等。
  - **任务调度请求**：在ISR中，如果发现有更高优先级的任务变为就绪状态，ISR会设置一个任务调度请求标志位（通常是 `xTaskSwitchRequired`）。
  ```c
  void vTimerISR(void) {
      // 处理定时器中断
      // ...

      // 检查是否有更高优先级的任务就绪
      if (xHigherPriorityTaskWoken) {
          // 设置任务调度请求标志位
          portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
      }
  }
  ```

- **中断返回**：
  - **检查调度请求**：在中断返回时，调度器会检查任务调度请求标志位。如果该标志位被设置，则进行上下文切换。
  - **上下文切换**：调用上下文切换函数（如 `portYIELD_FROM_ISR()`），进行上下文切换，将CPU使用权交给更高优先级的任务。
  ```c
  void vPortYieldFromISR(BaseType_t xSwitchRequired) {
      if (xSwitchRequired != pdFALSE) {
          // 保存当前任务的上下文
          portSAVE_CONTEXT();

          // 加载新任务的上下文
          portRESTORE_CONTEXT();
      }
  }
  ```

#### 4. 临界区保护

在任务切换过程中，某些关键操作需要在临界区内完成，以防止中断干扰。

- **进入临界区**：
  - **禁用中断**：通过调用 `taskENTER_CRITICAL()` 函数，禁用中断，确保临界区内的操作不会被中断打断。
  ```c
  taskENTER_CRITICAL();
  ```

- **退出临界区**：
  - **恢复中断**：通过调用 `taskEXIT_CRITICAL()` 函数，恢复中断，允许中断再次发生。
  ```c
  taskEXIT_CRITICAL();
  ```

### 调度和抢占的区别
> 调度是指操作系统决定哪一个任务应当运行的过程，而抢占则是指当一个更高优先级的任务变得可运行时，立即停止当前运行的任务，转而运行更高优先级的任务。

- _调度_：负责管理`CPU`的时间分配，决定哪些进程或任务可以获得`CPU`的使用权。分为抢占式调度和非抢占式调度。
- _抢占_：`FreeRTOS`通过设置**优先级**和使用**内核调度器**来实现抢占。当一个更高优先级的任务变为可运行状态时，调度器会立即触发上下文切换，保存当前任务的状态，恢复高优先级任务的状态
### 抢占式调度和非抢占式调度
- _非抢占式调度_：也称为非剥夺式调度，当前运行的任务必须主动放弃`CPU`，如任务运行完毕、遇到阻塞等。这种方式实现简单，系统开销小，但响应速度慢，不适合实时系统。
- _抢占式调度_：也称为剥夺式调度，允许高优先级任务**中断**低优先级任务的执行，获取`CPU`使用权。这种方式能够快速响应紧急任务，提高系统的实时性和响应效率。
### 中断是否会触发抢占
- _中断处理_：中断是由硬件触发的，用于处理外部事件（如I/O完成、定时器到期等）。中断处理程序会在中断上下文中执行，不会被视为一个任务。
- _中断与抢占_：中断处理过程中，如果中断处理程序使一个更高优先级的任务变为可运行状态，那么在中断处理完成后，调度器会检查是否有更高优先级的任务需要运行。如果有，当前任务会被抢占，CPU使用权会被分配给更高优先级的任务。
### FreeRTOS的主任务逻辑
> 通过一个**无限循环**实现，该循环负责任务调度，确保每个就绪的任务都能得到执行的机会。

1. _初始化_：在FreeRTOS启动时，首先会进行一系列的初始化工作，包括但不限于创建内核对象（如队列、信号量）、设置中断向量表、初始化系统时钟以及创建初始任务等。
2. _开始调度_：初始化完成后，调用`vTaskStartScheduler()`函数来开始任务调度器。此时，FreeRTOS进入多任务模式，开始执行第一个任务。
3. _任务切换_：FreeRTOS使用一个优先级轮转算法来进行任务调度。当高优先级任务变为可运行状态时，它会立即抢占当前正在运行的较低优先级任务。如果所有任务都在等待某个事件，则会运行空闲任务。
4. _无限循环_：FreeRTOS的核心是一个无限循环，这个循环不断地检查任务列表，寻找最高优先级的可运行任务，并将其投入执行。这个过程是通过调用任务调度器完成的。
5. _上下文切换_：当发生中断或任务自愿放弃CPU时，FreeRTOS会保存当前任务的状态，并恢复下一个最高优先级任务的状态，这一过程称为上下文切换。
6. _任务同步与通信_：FreeRTOS提供了多种机制用于任务间的同步和通信，如队列、信号量、互斥锁和事件组等。这些机制允许任务之间安全地共享资源或传递信息。
7. _空闲钩子函数_：FreeRTOS提供了一个空闲任务钩子函数（idle hook），可以在没有更高优先级任务可运行时执行特定操作，如降低功耗。
8. _系统时钟_：FreeRTOS依赖于一个周期性的系统时钟中断来驱动任务调度器，确保定时器功能和服务能够正常运作。

### FreeRTOS 如何放弃CPU

> 在FreeRTOS中，任务可以通过调用`taskYIELD()`、`vTaskDelay()`、`vTaskDelayUntil()`等函数来主动放弃`CPU`的使用权；或者通过**等待事件**（如队列接收、信号量获取等）来被动放弃`CPU`。
#### 主动放弃CPU

- _`taskYIELD()`_
  - _功能_：当前任务主动放弃剩余的时间片，让调度器选择下一个最高优先级的可运行任务。
  - _底层实现_：
    ```c
    void taskYIELD( void ) {
        portYIELD();
    }
    ```
    `portYIELD()` 是一个汇编级别的函数，用于**触发上下文切换**。它通常会设置一个标志位，使得下次中断返回时进行任务切换。

- _`vTaskDelay()`_
  - _功能_：当前任务延迟一段时间，期间任务会进入阻塞状态，释放CPU使用权。
  - _底层实现_：
    ```c
    void vTaskDelay( TickType_t xTicksToDelay ) {
        vTaskDelayUntil( NULL, xTicksToDelay );
    }
    ```
    `vTaskDelayUntil` 会更新任务的延迟时间，并将任务状态设置为阻塞。调度器会在下一个**时钟中断**时检查任务状态，如果延迟时间已到，任务会恢复为就绪状态。

- _`vTaskDelayUntil()`_
  - _功能_：任务在固定的时间间隔内周期性地执行，确保每次_执行的时间间隔_保持一致。
  - _底层实现_：
    ```c
    BaseType_t xTaskDelayUntil( TickType_t *pxPreviousWakeTime, const TickType_t xTimePeriod ) {
        TickType_t xTimeToWake;
        
        xTimeToWake = *pxPreviousWakeTime + xTimePeriod;
        vTaskSetTimeOutState( &xTimeOut );
        
        while( xTaskCheckForTimeOut( &xTimeOut, &xTimeToWake ) == pdFALSE ) {
            vTaskDelay( xTimeToWake - xTaskGetTickCount() );
        }
        
        *pxPreviousWakeTime = xTimeToWake;
        return xTimeOut.xOverflowed;
    }
    ```
    这个函数会计算当前时间与上次唤醒时间的差值，并根据指定的频率计算出剩余的延时时间。任务进入阻塞状态，直到延时时间到达，然后恢复为就绪状态。
#### 被动放弃CPU
> 等待事件

  - _队列接收 (`xQueueReceive()`)_
    - _功能_：从队列中接收数据，如果队列为空，任务会进入阻塞状态，等待数据可用。
    - _底层实现_：
      ```c
      BaseType_t xQueueReceive( QueueHandle_t xQueue, void *pvBuffer, TickType_t xTicksToWait ) {
          return prvReceive( xQueue, pvBuffer, xTicksToWait, pdFALSE );
      }

      static BaseType_t prvReceive( QueueHandle_t xQueue, void *pvBuffer, TickType_t xTicksToWait, BaseType_t xCopyPosition ) {
          // 检查队列是否有数据
          if (xQueue->uxMessagesWaiting > 0) {
              // 数据可用，拷贝数据并返回
              queueCOPY_DATA_FROM_QUEUE( xQueue, pvBuffer, xCopyPosition );
              return pdPASS;
          } else {
              // 队列为空，任务进入阻塞状态
              vTaskSuspendAll();
              traceQUEUE_RECEIVE( xQueue, xCopyPosition );
              prvLockQueue( xQueue );
              prvAddCurrentTaskToDelayedList( xTicksToWait );
              prvUnlockQueue( xQueue );
              ( void ) xQueueGenericSend( xQueue, NULL, 0U, queueSEND_TO_BACK );
              traceQUEUE_RECEIVE_FROM_ISR( xQueue, xCopyPosition );
              ( void ) xTaskResumeAll();
              return pdFAIL;
          }
      }
      ```
      如果队列中有数据，任务会**拷贝**数据并返回；如果队列为空，任务会进入阻塞状态，等待数据可用。

  - _信号量获取 (`xSemaphoreTake()`)_
    - _功能_：获取一个信号量，如果信号量不可用，任务会进入阻塞状态，等待信号量可用。
    - _底层实现_：
      ```c
      BaseType_t xSemaphoreTake( SemaphoreHandle_t xSemaphore, TickType_t xBlockTime ) {
          return xQueueSemaphoreTake( xSemaphore, xBlockTime );
      }

      static BaseType_t xQueueSemaphoreTake( QueueHandle_t xQueue, TickType_t xTicksToWait ) {
          // 检查信号量是否可用
          if (xQueue->uxMessagesWaiting > 0) {
              // 信号量可用，减少计数并返回
              --( xQueue->uxMessagesWaiting );
              return pdTRUE;
          } else {
              // 信号量不可用，任务进入阻塞状态
              vTaskSuspendAll();
              traceQUEUE_SEMAPHORE_take( xQueue );
              prvLockQueue( xQueue );
              prvAddCurrentTaskToDelayedList( xTicksToWait );
              prvUnlockQueue( xQueue );
              ( void ) xQueueGenericSend( xQueue, NULL, 0U, queueSEND_TO_BACK );
              traceQUEUE_SEMAPHORE_take_FROM_ISR( xQueue );
              ( void ) xTaskResumeAll();
              return pdFALSE;
          }
      }
      ```
      如果信号量可用，任务会减少信号量计数并返回；如果信号量不可用，任务会进入阻塞状态，等待信号量可用。
### FreeRTOS任务怎么分的，优先级FreeRTOS怎么识别的？
> 在 `FreeRTOS` 中，任务的管理与调度是基于**优先级**机制。
#### 任务分类
FreeRTOS 支持创建多个任务，每个任务可以被赋予一个优先级。优先级的设定允许系统根据任务的重要程度来调度任务的执行。FreeRTOS 中的任务可以根据其优先级和状态来分类：

- **按优先级分类**：每个任务都有一个与之关联的优先级，优先级的数值越小，代表任务的优先级越高。优先级的范围通常是从 `0` 到 `configMAX_PRIORITIES - 1`，其中 `configMAX_PRIORITIES` 是一个在 `FreeRTOSConfig.h` 文件中定义的宏，用于指定系统中可以使用的最大优先级数量。例如，如果 `configMAX_PRIORITIES` 设置为 `5`，则用户可以使用的优先级编号为 `0` 至 `4` 。
- **按状态分类**：FreeRTOS 中的任务可以处于四种不同的状态：运行态（Running）、就绪态（Ready）、阻塞态（Blocked）和挂起态（Suspended）。这些状态反映了任务当前是否具备运行条件，以及何时能够再次获得 CPU 控制权。
#### 优先级识别
FreeRTOS 的调度器使用任务优先级来决定哪个任务应该被执行。优先级识别的过程可以通过以下几种方式实现：

- **通用方法**：使用 C 语言编写的通用方法，这种方法适用于所有架构，但是效率相对较低。通用方法会在每次任务切换时遍历所有就绪任务的优先级列表，找到优先级最高的任务来执行。这种方法对 `configMAX_PRIORITIES` 的取值没有限制，但建议尽量小以减少内存消耗和提高效率 。
- **优化方法**：对于某些架构，FreeRTOS 提供了优化过的优先级识别方法。这些方法利用了特定架构的特性，如 ARM Cortex-M 架构提供的 `CLZ`（`Count Leading Zeros`）指令，可以快速找到优先级列表中优先级最高的任务。使用优化方法时，`configMAX_PRIORITIES` 的取值通常不能超过 `32`。要启用优化方法，需要在 `FreeRTOSConfig.h` 文件中定义 `configUSE_PORT_OPTIMISED_TASK_SELECTION` 为 `1` 。

### 任务参数
- 单个参数
	空指针：更方便复用
	类型指针

- 多个参数
	传递结构体指针
	复用任务创建函数
### 状态
#### Running
#### Ready
- `xTaskCreate`
---
#### Blocked
- `vTaskdelay`
- **阻塞**：因为等待某个资源（如I/O操作，信号量，消息队列等）而无法继续执行。
#### Suspended
- `vTaskSuspend()`
- **挂起**：将一个正在运行的任务从活动状态转换为挂起状态。与阻塞的区别是不需要等待某个资源是否可用，可通过API函数直接控制。
---
### 任务优先级
- 数字越大优先级越高 0~24
- 高等级任务不进入阻塞或挂起状态，低等级任务将永远无法执行
### Watchdog
1. 注册看门狗
2. 缺省时间 `5s`
3. 喂狗
- 看门狗监控所注册的任务，每一个内核有一个看门狗
- FreeRTOS 任务`IDLE`,优先级0，**清除残存的任务，释放内存**
---
- 即使有多个任务，只要有一个任务在看门狗触发重启之前成功地重置了看门狗，那么系统就不会重启。
- 没有任何单个任务的连续运行时间会比看门狗的重启时间长。
## 数据结构
### queue
- FIFO
- 写入和读取的大小格式相同
#### 单种数据
- 创建队列
- 消费数据
#### 多种数据
- 用结构体存储信息
- 创建id来区分信息的不同来源
### Stream Buffer
- FIFO
- `Stream Buffer`读写的大小没有限制
- 受众对象就是 流数据 比如MP3，视频，在线电台等
### Message Buffer
- 由`string buffer`实现，相对于`string buffer`自带`4`个字节的信息，来告知接收端本次信息的总长度
- 输入和输出的类型大小相同，但每一次的数据可以有不同的类型大小
### 同优先级的任务调度通过链表实现
在FreeRTOS中，同优先级的任务调度是通过链表来实现的。每个优先级对应一个链表，链表中的节点表示同优先级的任务。当需要调度任务时，FreeRTOS会从链表头部取出任务并执行。
#### 任务控制块（TCB）

每个任务都有一个任务控制块（TCB），其中包含任务的各种信息，如任务函数、任务名称、优先级、堆栈指针等。

```c
typedef struct TCB {
    StackType_t *pxTopOfStack;  // 任务堆栈的顶部指针
    List_t xStateList;          // 任务状态列表项
    UBaseType_t uxPriority;     // 任务优先级
    char pcTaskName[configMAX_TASK_NAME_LEN];  // 任务名称
    // 其他任务相关信息
} TCB_t;
```
#### 任务列表

FreeRTOS使用链表来管理任务列表。每个优先级对应一个链表，链表中的节点表示同优先级的任务。

```c
typedef struct List {
    ListItem_t *pxIndex;         // 指向链表的索引
    UBaseType_t uxNumberOfItems; // 链表中项目的数量
    const char *pcName;          // 链表名称
} List_t;

typedef struct ListItem {
    struct ListItem *pxNext;     // 指向下一个节点
    struct ListItem *pxPrevious; // 指向前一个节点
    TickType_t xItemValue;       // 节点值
    void *pvOwner;               // 节点所属的任务
} ListItem_t;
```

#### 选择最高优先级的就绪任务列表

FreeRTOS通过遍历就绪任务列表，找到最高优先级的非空列表。

```c
BaseType_t xTaskIncrementTick(void) {
    // 增加系统滴答计数器
    xTickCount++;

    // 检查延迟任务列表
    if (xTickCount >= xNextTaskUnblockTime) {
        // 将延迟任务移到就绪任务列表
        prvCheckDelayedList();
    }

    // 选择最高优先级的就绪任务列表
    UBaseType_t uxTopReadyPriority = (UBaseType_t)uxCurrentReadyPriority;
    if (uxTopReadyPriority > (UBaseType_t)tskIDLE_PRIORITY) {
        // 有更高优先级的任务就绪
        if (uxSchedulerSuspended == pdFALSE) {
            // 设置任务调度请求标志位
            traceINCREASE_TICK(xTickCount);
            portYIELD_WITHIN_API();
        } else {
            // 记录需要调度的任务
            xYieldPending = pdTRUE;
        }
    }

    return pdFALSE;
}
```
#### 从列表头部取出任务

FreeRTOS从最高优先级的就绪任务列表中取出任务，并将其标记为当前运行的任务。

```c
static void prvTaskExitError(void) {
    // 错误处理
    for (;;);
}

void vTaskSwitchContext(void) {
    // 保存当前任务的上下文
    portSAVE_CONTEXT();

    // 选择下一个任务
    TCB_t *pxNextTCB = NULL;
    UBaseType_t uxTopReadyPriority = (UBaseType_t)uxCurrentReadyPriority;

    // 查找最高优先级的就绪任务
    while ((pxNextTCB = (TCB_t *)listGET_OWNER_OF_NEXT_ENTRY((ListItem_t *)&xNullItem, &(xReadyTasksLists[uxTopReadyPriority]))) == NULL) {
        --uxTopReadyPriority;
    }

    // 更新当前任务
    pxCurrentTCB = pxNextTCB;

    // 恢复新任务的上下文
    portRESTORE_CONTEXT();
}
```
## 线程安全
### 原子变量的特点
1. **线程安全**：原子操作保证了在多线程环境中操作的完整性，不会被其他线程中断。
2. **简单高效**：原子变量通常用于简单的同步操作，如计数器、标志位等。
3. **无阻塞**：原子操作不会阻塞当前线程，因此不会直接导致死锁。
### 原子变量不会直接引起死锁的原因

- **原子操作的特性**：原子操作是不可分割的，不会被其他线程中断，因此不会出现两个线程同时争夺同一个资源的情况。
- **无阻塞**：原子操作不会阻塞当前线程，即使多个线程同时访问同一个原子变量，也不会导致任何一个线程被永久阻塞。
### 原子变量可能导致死锁的间接情况

尽管原子变量本身不会引起死锁，但在复杂的并发环境中，不当使用原子变量结合其他同步机制时，可能会导致死锁。以下是一些常见的情景：

1. **复合操作**：如果原子变量用于控制某个复杂操作的执行，而该操作又涉及其他同步机制（如锁），则可能引入死锁风险。

   ```c
   std::atomic<bool> flag(false);
   std::mutex mutex;

   void thread1() {
       while (!flag.load()) {
           // 等待 flag 变为 true
       }
       std::lock_guard<std::mutex> lock(mutex);
       // 执行某些操作
   }

   void thread2() {
       std::lock_guard<std::mutex> lock(mutex);
       // 执行某些操作
       flag.store(true);
   }
   ```

   在这个例子中，如果 `thread2` 在获取锁后被抢占，而 `thread1` 在等待 `flag` 变为 `true` 时不断忙等，可能会导致 `thread2` 无法继续执行，从而导致死锁。

2. **锁的顺序**：如果多个线程在不同顺序上获取多个锁，可能会导致死锁。

   ```c
   std::atomic<bool> flag(false);
   std::mutex mutex1;
   std::mutex mutex2;

   void thread1() {
       std::lock_guard<std::mutex> lock1(mutex1);
       if (flag.load()) {
           std::lock_guard<std::mutex> lock2(mutex2);
           // 执行某些操作
       }
   }

   void thread2() {
       std::lock_guard<std::mutex> lock2(mutex2);
       flag.store(true);
       std::lock_guard<std::mutex> lock1(mutex1);
       // 执行某些操作
   }
   ```

   在这个例子中，如果 `thread1` 先获取 `mutex1` 并等待 `flag` 变为 `true`，而 `thread2` 先获取 `mutex2` 并将 `flag` 设置为 `true`，然后尝试获取 `mutex1`，则可能会导致死锁。

3. **条件变量**：如果原子变量用于控制条件变量的等待和通知，不当使用也可能导致死锁。

   ```c
   std::atomic<bool> flag(false);
   std::mutex mutex;
   std::condition_variable cv;

   void thread1() {
       std::unique_lock<std::mutex> lock(mutex);
       while (!flag.load()) {
           cv.wait(lock);
       }
       // 执行某些操作
   }

   void thread2() {
       std::unique_lock<std::mutex> lock(mutex);
       flag.store(true);
       cv.notify_one();
   }
   ```

   在这个例子中，如果 `thread1` 在等待 `flag` 变为 `true` 时被阻塞，而 `thread2` 在获取锁后将 `flag` 设置为 `true` 并通知 `cv`，则 `thread1` 会被唤醒并继续执行。但如果 `thread2` 在通知 `cv` 后被抢占，而 `thread1` 在等待 `flag` 变为 `true` 时不断忙等，可能会导致死锁。
### 在FreeRTOS中如何保证原子性操作
在FreeRTOS中，保证原子性操作主要通过以下几种方式实现：使用硬件提供的原子指令、内存屏障、临界区（`Critical Sections`）和任务调度禁用（`Scheduler Suspension`）。下面详细解释这些方法及其在FreeRTOS中的实现。

#### 1. 硬件提供的原子指令

FreeRTOS可以利用底层硬件提供的原子指令来实现原子操作。这些指令通常在特定的处理器架构中提供，确保操作的原子性。

ARM Cortex-M架构提供了 `LDREX`（Load Exclusive）和 `STREX`（Store Exclusive）指令，用于实现原子的读取和写入操作。
```c
volatile uint32_t shared_var = 0;

bool atomic_update(uint32_t new_value) {
    uint32_t old_value;
    do {
        old_value = __LDREXW(&shared_var);  // 原子读取
    } while (__STREXW(new_value, &shared_var) != 0);  // 原子写入
    return true;
}
```

#### 2. FreeRTOS中的内存屏障
```c
__asm volatile ("dmb" : : : "memory");  // ARM 架构中的全内存屏障
```

#### 3. FreeRTOS中的临界区
```c
void critical_section_example(void) {
    taskENTER_CRITICAL();  // 进入临界区
    // 临界区内的代码
    shared_var++;
    taskEXIT_CRITICAL();   // 退出临界区
}
```

#### 4. 任务调度禁用
任务调度禁用可以暂时停止任务调度，确保当前任务独占CPU。FreeRTOS提供了 `vTaskSuspendAll` 和 `xTaskResumeAll` 函数来实现任务调度禁用。
```c
void scheduler_suspension_example(void) {
    vTaskSuspendAll();  // 禁用任务调度
    // 临界区内的代码
    shared_var++;
    xTaskResumeAll();   // 恢复任务调度
}
```

#### 5. 使用 `portENTER_CRITICAL` 和 `portEXIT_CRITICAL`
FreeRTOS提供了一组宏 `portENTER_CRITICAL` 和 `portEXIT_CRITICAL`，用于进入和退出临界区。这些宏在不同的硬件平台上会有不同的实现，确保了跨平台的兼容性。
```c
void port_critical_section_example(void) {
    portENTER_CRITICAL();  // 进入临界区
    // 临界区内的代码
    shared_var++;
    portEXIT_CRITICAL();   // 退出临界区
}
```
### 临界区的内部实现

在FreeRTOS中，临界区的实现通常涉及到关闭中断和恢复中断。这可以通过硬件提供的中断控制寄存器来实现。

```c
// FreeRTOS port 层的实现
UBaseType_t ulPortSetInterruptMask(void) {
    UBaseType_t ulOriginalPSR;
    __asm volatile (
        "MRS %0, PRIMASK\n"  // 读取当前的 PRIMASK 寄存器
        "CPSID i\n"          // 关闭中断
        : "=r"(ulOriginalPSR)
    );
    return ulOriginalPSR;
}

void vPortClearInterruptMask(UBaseType_t ulPSR) {
    __asm volatile (
        "MSR PRIMASK, %0\n"  // 恢复 PRIMASK 寄存器
        :
        : "r"(ulPSR)
    );
}
```

#### `portENTER_CRITICAL` 和 `portEXIT_CRITICAL` 的实现

```c
#define portENTER_CRITICAL() ulPortSetInterruptMask()
#define portEXIT_CRITICAL() vPortClearInterruptMask(0)
```
### 信号量与互斥锁

1. **数据结构**：
   - **信号量**：相对简单，主要包括计数值和内部队列。
   - **互斥量**：更复杂，增加了所有权管理和递归计数。

2. **优先级继承**：
   - **信号量**：不支持优先级继承。
   - **互斥量**：支持优先级继承，防止优先级反转问题。

3. **所有权管理**：
   - **信号量**：没有所有权的概念，任何任务都可以释放信号量。
   - **互斥量**：具有所有权的概念，只有拥有互斥量的任务才能释放它。

4. **递归调用**：
   - **信号量**：不支持递归调用。
   - **互斥量**：支持递归调用，递归计数会增加。
#### 死锁与优先级反转的区别
> 死锁（Deadlock）
- **定义**：多个进程在执行过程中，因争夺资源而陷入的一种僵局，每个进程都持有一些资源并等待其他进程释放它们持有的资源，导致所有进程都无法继续执行。
- **涉及**：多个进程之间的资源争夺。
- **条件**：互斥、占有和等待、不可剥夺、循环等待。
- **影响**：所有涉及的进程都会停滞不前，直到外部干预。
- **解决方法**：预防、避免、检测和解除死锁。

> 优先级反转（Priority Inversion）
- **定义**：在实时系统中，一个高优先级任务等待一个低优先级任务持有的资源时，如果中等优先级任务介入并抢占了低优先级任务，导致高优先级任务被延迟。
- **涉及**：任务优先级和资源访问。
- **条件**：高优先级任务等待低优先级任务持有的资源，中等优先级任务介入。
- **影响**：可能导致高优先级任务错过执行的截止时间，影响实时性。
- **解决方法**：优先级继承和优先级天花板协议。

> 主要区别
- **问题领域**：死锁是多任务环境中的普遍问题，而优先级反转是实时系统中的特定问题。
- **发生条件**：死锁需要多个进程相互等待资源，优先级反转是高优先级任务等待低优先级任务持有的资源且有中等优先级任务介入。
- **解决方法**：死锁的解决方法侧重于避免、检测和解决死锁状态，优先级反转的解决方法侧重于调整任务优先级以保证高优先级任务的及时执行。
### 信号量（Semaphore）

#### 数据结构
信号量的数据结构相对简单，主要包括一个计数值和一些同步原语。

```c
typedef struct xSEMAPHORE {
    volatile UBaseType_t uxRecursiveCallCount;  // 递归调用计数
    volatile UBaseType_t uxCurrentOwners;       // 当前所有者
    struct xQUEUE xQueue;                       // 内部队列
} Semaphore_t;
```

#### 创建信号量
创建信号量时，会初始化一个内部队列，用于管理等待任务。

```c
SemaphoreHandle_t xSemaphoreCreateBinary(void) {
    Queue_t *pxNewSemaphore;
    pxNewSemaphore = xQueueGenericCreateInternal((UBaseType_t)1, (UBaseType_t)sizeof(BaseType_t), queueQUEUE_TYPE_BINARY_SEMAPHORE, true);
    if (pxNewSemaphore != NULL) {
        pxNewSemaphore->uxLength = (UBaseType_t)1;
        pxNewSemaphore->uxItemSize = (UBaseType_t)sizeof(BaseType_t);
        pxNewSemaphore->ucQueueType = queueQUEUE_TYPE_BINARY_SEMAPHORE;
        pxNewSemaphore->uxRecursiveCallCount = (UBaseType_t)0;
        pxNewSemaphore->uxCurrentOwners = (UBaseType_t)0;
    }
    return pxNewSemaphore;
}
```

#### 获取信号量
获取信号量时，会检查信号量是否可用。如果不可用，任务会被挂起到等待队列中。

```c
BaseType_t xSemaphoreTake(SemaphoreHandle_t xSemaphore, TickType_t xTicksToWait) {
    return xQueueSemaphoreTake((Queue_t *)xSemaphore, xTicksToWait);
}
```

#### 释放信号量
释放信号量时，会增加信号量的计数值，并唤醒等待队列中的任务。

```c
BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore) {
    return xQueueGenericSend((Queue_t *)xSemaphore, (const void *)pdPASS, 0, queueSEND_TO_BACK);
}
```

### 互斥量（Mutex）
#### 数据结构
互斥量的数据结构比信号量更复杂，增加了优先级继承的支持和所有权管理。

```c
typedef struct xMUTEX {
    volatile UBaseType_t uxRecursiveCallCount;  // 递归调用计数
    volatile UBaseType_t uxOwner;               // 当前所有者
    struct xQUEUE xQueue;                       // 内部队列
} Mutex_t;
```

#### 创建互斥量
创建互斥量时，会初始化一个内部队列，并设置互斥量的类型。

```c
SemaphoreHandle_t xSemaphoreCreateMutex(void) {
    Queue_t *pxNewMutex;
    pxNewMutex = xQueueGenericCreateInternal((UBaseType_t)1, (UBaseType_t)sizeof(BaseType_t), queueQUEUE_TYPE_MUTEX, true);
    if (pxNewMutex != NULL) {
        pxNewMutex->uxLength = (UBaseType_t)1;
        pxNewMutex->uxItemSize = (UBaseType_t)sizeof(BaseType_t);
        pxNewMutex->ucQueueType = queueQUEUE_TYPE_MUTEX;
        pxNewMutex->uxRecursiveCallCount = (UBaseType_t)0;
        pxNewMutex->uxOwner = (UBaseType_t)0;
    }
    return pxNewMutex;
}
```

#### 获取互斥量
获取互斥量时，会检查互斥量是否可用，并管理所有权。如果当前任务已经拥有互斥量，递归计数会增加。

```c
BaseType_t xSemaphoreTake(SemaphoreHandle_t xSemaphore, TickType_t xTicksToWait) {
    Queue_t *pxQueue = (Queue_t *)xSemaphore;
    TCB_t *pxCurrentTCB = (TCB_t *)pxCurrentTCB;

    if (pxQueue->ucQueueType == queueQUEUE_TYPE_MUTEX) {
        if (pxQueue->uxOwner == (UBaseType_t)pxCurrentTCB) {
            // 已经拥有互斥量，递归计数增加
            pxQueue->uxRecursiveCallCount++;
            return pdPASS;
        } else {
            // 尝试获取互斥量
            if (xQueueReceive(pxQueue, NULL, xTicksToWait) == pdPASS) {
                pxQueue->uxOwner = (UBaseType_t)pxCurrentTCB;
                pxQueue->uxRecursiveCallCount = 1;
                return pdPASS;
            } else {
                return pdFAIL;
            }
        }
    } else {
        return xQueueSemaphoreTake(pxQueue, xTicksToWait);
    }
}
```

#### 释放互斥量
释放互斥量时，会减少递归计数。如果递归计数为零，互斥量会被释放，并唤醒等待队列中的任务。此外，还会处理优先级继承。

```c
BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore) {
    Queue_t *pxQueue = (Queue_t *)xSemaphore;
    TCB_t *pxCurrentTCB = (TCB_t *)pxCurrentTCB;

    if (pxQueue->ucQueueType == queueQUEUE_TYPE_MUTEX) {
        if (pxQueue->uxOwner == (UBaseType_t)pxCurrentTCB) {
            // 递归计数减少
            if (--pxQueue->uxRecursiveCallCount == 0) {
                // 递归计数为零，释放互斥量
                pxQueue->uxOwner = 0;
                return xQueueGenericSend(pxQueue, (const void *)pdPASS, 0, queueSEND_TO_BACK);
            } else {
                return pdPASS;
            }
        } else {
            // 当前任务不是互斥量的所有者
            return pdFAIL;
        }
    } else {
        return xQueueGenericSend(pxQueue, (const void *)pdPASS, 0, queueSEND_TO_BACK);
    }
}
```
### Semaphore
- 仅仅是内存中的一个整数（`int`)
- consumer---`take`-1
- producer---`give`+1
- 线程是在信号量上排队等待的，阻塞，效果是线程不会被执行，会被挂起，只有信号量有了或者超时时间到了才会执行一次。
### binary semaphore
- 二进制信号量表面上等价于定义一个全局变量`leap`，实际上信号量并不会让CPU一直判断是否改变，而是通过阻塞任务来释放CPU资源。
### counting semaphore
- `Resoures`控制
- `Event`控制
### Event Group
- 3`bytes`--24`bit`
### Direct Task Notification
- 取代二进制信号量，计数型信号量，消息组。更少内存，更快速度
- 一个任务可以有n个通知，每个通知有一个`value`和`status`，操作对象为`value`
- `value`：四字节
- `status`：一字节
## 进阶
- [x] FreeRTOS Docs
- [ ] Developer Docs
- [ ] API Reference
- [ ] github_orgin_code
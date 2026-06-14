## Preface

| IRQ | Interrupt  Request        |
| --- | ------------------------- |
| ISR | Interrupt Service Routine |
| SIL | Safety Integrity Level    |
| TCB | Task Control Block        |
In FreeRTOS, each thread of execution is called a 'task'.
### Why Use an RTOS?
> task prioritization can help ensure an application meets its processing deadlines, but a kernel can bring other less obvious benefits.

- *Abstracting away timing information*：RTOS负责计时，为app提供对应时间API，应用程序的结构更加简单。
- *Maintainability/Extensibility*：抽象出`timing`后内核负责计时，解耦app与硬件设备
- *Modularity*：每个任务是独立的模块
- *Team development*:Tasks should also have well-defined interfaces, allowing easier team development.
- *Easier testing*
- *Code reuse*:Code designed with greater modularity and fewer interdependencies is easier to reuse.
- *Improved efficiency*
- *Idle time*:The Idle task can measure spare processing capacity, perform background checks, or place the processor into a low-power mode.
- *Power Management*:The efficiency gains that result from using an RTOS allow the processor to spend more time in a low power mode.
- *Flexible interrupt*: handling Interrupt handlers can be kept very short by deferring processing to either a task created by the application writer or the automatically created RTOS daemon task (also known as the timer task).
- *Mixed processing requirements*: Simple design patterns can achieve a mix of periodic, continuous, and event-driven processing within an application. In addition, hard and soft real-time requirements can be met by selecting appropriate task and interrupt priorities.
### FreeRTOS Kernel Features
- *Pre-emptive or co-operative operation*
- Optional time-slicing Very flexible task priority assignment
- Flexible, fast and light-weight task notification mechanisms
- Queues
- Binary semaphores
- Counting semaphores
- Mutexes
- Recursive mutexes
- Software timers
- Event group
- Stream buffers
- Message buffers
- Tick hook functions
- Idle hook functions
- Stack overflow checking Trace macros
- Task run-time statistics gathering
- Optional commercial licensing and support
- Full interrupt nesting model (for some architectures)
- A tick-less capability for extreme low power applications (for some architectures)
- Memory Protection Unit support for isolating tasks and increasing application safety (for some architectures)
- Software managed interrupt stack when appropriate (this can help save RAM) The ability to create RTOS objects using either statically or dynamically allocated memory
## The FreeRTOS Kernel Distribution
### FreeRTOS Source Files Common to All Ports
> `tasks.c` and `list.c` implement the core FreeRTOS kernel functionality and `are always required`. They are located directly in the FreeRTOS/Source directory
```
FreeRTOS
└── Source
    ├── tasks.c          # Core implementation for task management, always required
    ├── list.c           # Core implementation for list management, used for task queues, always required
    ├── queue.c          # queue and semaphore services, nearly always required
    ├── timers.c         # software timer functionality, optional and used as needed
    ├── event_groups.c   # event group functionality, optional and used as needed
    ├── stream_buffer.c  # stream buffer and message buffer functionality, optional and used as needed
    └── croutine.c       x Implementation for coroutine support, optional and no longer maintained
```

> A source file that uses the FreeRTOS API must include `FreeRTOS.h`,
> Do not explicitly include any other FreeRTOS header files—FreeRTOS.h automatically includes `FreeRTOSConfig.h`.

### Data Types and Coding Style Guide
#### Data Types
> Each port of FreeRTOS has a unique portmacro.h header file that contains (amongst other things) definitions for two port-specific data types: `TickType_t` and `BaseType_t`.
- *TickType_t*:TickType_t is the data type used to hold the tick count value, and to specify times.
- *BaseType_t*:This is always defined as the most efficient data type for the `architecture`.BaseType_t is generally used for `return types` that take only a very `limited range of values`, and for pdTRUE/pdFALSE type Booleans.
#### Function Names
- *vTaskPrioritySet()* returns a void and is defined within `tasks.c`.
- *xQueueReceive()* returns a variable of type BaseType_t and is defined within `queue.c`.
- *pvTimerGetTimerID()* returns a `pointer to void` and is defined within` timers.c`.`
#### Macro Names
> The `semaphore` API is written almost entirely as a set of macros, but follows the function naming convention, rather than the macro naming convention.

| Prefix | Example Macro Definition | Header File               |
| ------ | ------------------------ | ------------------------- |
| port   | portMAX_DELAY            | portable.h or portmacro.h |
| task   | taskENTER_CRITICAL()     | task.h                    |
| pd     | pdTRUE                   | projdefs.h                |
| config | configUSE_PREEMPTION     | FreeRTOSConfig.h          |
| err    | errQUEUE_FULL            | projdefs.h                |

| Macro Definition | Macro Value |
| ---------------- | ----------- |
| pdTRUE           | 1           |
| pdFALSE          | 0           |
| pdPASS           | 1           |
| pdFAIL           | 0           |
## Heap Memory Management
### Static and Dynamic Memory Allocation
- *Kernel objects* such as tasks, queues, semaphores, and event groups. The RAM required to hold these objects can be allocated *statically* at `compile-time` or *dynamically* at `run time`.
- *Dynamic allocation* reduces design and planning effort, `simplifies` the API, and `minimizes` the RAM footprint.
- *Static allocation* is more `deterministic`, removes the need to handle memory allocation failures, and removes the `risk` of heap fragmentation (where the heap has enough free memory but not in one usable contiguous block).
### Libc malloc
> Dynamic memory allocation is a C programming concept.
> The general-purpose C library malloc() and free() functions may not be suitable for one or more of the following reasons:
- They are not always available on *small* embedded systems.
- They are rarely *thread-safe*.
- They are not *deterministic*; the amount of time taken to execute the functions will differ from call to call.
- They can suffer from *fragmentation*.
- They can complicate the *linker configuration*.
### Options for Dynamic Memory Allocation
> pvPortMalloc() has the same prototype as the standard C library malloc() function, and vPortFree() has the same prototype as the standard C library free() function.

### Example Memory Allocation Schemes
#### Heap_1
> It is common for small, dedicated embedded systems to only create tasks and other `kernel objects` before starting the FreeRTOS scheduler.

- Heap_1.c implements a very basic version of `pvPortMalloc()`, and does not implement `vPortFree()`.
- Critical systems often prohibit dynamic memory allocatio because of *the uncertainties* associated with `non-determinism`, `memory fragmentation`, and `failed allocations`. Heap_1 is always *deterministic* and cannot fragment memory.
- Heap_1's implementation of `pvPortMalloc()` simply subdivides *a simple uint8_t array* called the `FreeRTOS heap` into smaller blocks each time it's called.
- *FreeRTOS appear to consume a lot of RAM*:implementing the heap as a statically allocated array so the heap becomes part of the FreeRTOS data.
- Each dynamically allocated task results in *two calls*to pvPortMalloc(). The first allocates a task control block (`TCB`), and the second the task's `stack`.
#todo 图示：Figure3.1 RAM being

#### Heap_2
> Heap_2 is superseded by `heap_4`, which includes enhanced functionality. Heap_2 is kept in the FreeRTOS distribution for backward compatibility and is **not** recommended for `new designs`.

- Heap_2.c also works by subdividing an **array** `dimensioned` by the configTOTAL_HEAP_SIZE constant.
- *FreeRTOS appear to consume a lot of RAM*:Implementing the heap as a statically allocated array so the heap becomes part of the FreeRTOS data.
---
- ==The best-fit algorithm== ensures that `pvPortMalloc()` uses the free block of memory that is *closest in size* to the number of bytes requested.
- Unlike heap_4, heap_2 does *notcombine adjacent free blocks* into a single larger block, so it is more susceptible to fragmentation than heap_4.
- If the size of the stack allocated to the newly created task is the *same size* as that allocated to `the previously deleted task`, then `the best-fit algorithm` reuses the block of RAM that held the stack of the deleted task to `hold` the stack of the created task.
#todo 图示：Figure3.2 RAM being
#### Heap_3
> Heap_3.c uses `the standard library` `malloc()` and `free()` functions, so `the linker configuration` defines `the heap size`, and the configTOTAL_HEAP_SIZE constant is not used.

- Heap_3 makes `malloc()` and `free()` *thread-safe* by *temporarily suspending the FreeRTOS scheduler* for the duration of their execution.
#### Heap_4
> Like `heap_1` and `heap_2`, `heap_4` works by subdividing an array into `smaller blocks`. As before, the array is `statically allocated` and `dimensioned` by configTOTAL_HEAP_SIZE

- Heap_4 uses ==a first-fit algorithm== to allocate memory. Unlike heap_2, heap_4 *combines (coalesces) adjacent free blocks* of memory into *a single larger block*, which minimizes the risk of memory fragmentation.
- Making it suitable for applications that *repeatedly allocate* and free *different-sized* blocks of RAM.
#todo 图示：Figure3.2 RAM being
- Heap_4 is `not deterministic` but is *faster* than most standard library implementations of malloc() and free().
#### Heap_5
> Heap_5 uses `the same allocation algorithm` as heap_4. Unlike heap_4, which is limited to allocating memory from a single array, heap_5 can combine memory from **multiple separated memory spaces** into a single heap.

- Heap_5 is useful when `the RAM` provided by the system on which FreeRTOS is running does `not appear as a single contiguous` (without space) block in the system's memory map.
#### Initialising heap_5: The vPortDefineHeapRegions() API Function
> Heap_5 is the `only` provided heap allocation scheme that requires **explicit initialisation** and can't be used until after the call to `vPortDefineHeapRegions()`.

- *vPortDefineHeapRegions()* initialises heap_5 by specifying `the start address` and `size of each separate memory area` that makes up the heap managed by heap_5.
- That means kernel objects, such as tasks, queues, and semaphores, cannot be created dynamically until after the call to *vPortDefineHeapRegions()*.
```C
/**
 * @brief Defines memory regions for the heap.
 *
 * This function is used to initialize one or more blocks of memory that will be
 * part of the heap for the RTOS tasks. Each region is defined by its starting
 * address and its size, and these regions can be non-contiguous.
 *
 * @param[in] pxHeapRegions Pointer to an array containing descriptors of all
 *                          memory regions.
 */
void vPortDefineHeapRegions( const HeapRegion_t * const pxHeapRegions );


/**
 * @brief Describes a region of memory that will be part of the heap.
 *
 * @var pucStartAddress The start address of the memory region.
 * @var xSizeInBytes The size of the memory region in bytes.
 */
typedef struct HeapRegion
{
    /** The start address of the memory region. */
    uint8_t *pucStartAddress;

    /** The size of the memory region in bytes. */
    size_t xSizeInBytes;
} HeapRegion_t;

/***************************************************************************/
/**
 * @brief Defines the start address and size of the three RAM regions.
 */
#define RAM1_START_ADDRESS ( (uint8_t *) 0x00010000 )
#define RAM1_SIZE          ( 64 * 1024 )
#define RAM2_START_ADDRESS ( (uint8_t *) 0x00020000 )
#define RAM2_SIZE          ( 32 * 1024 )
#define RAM3_START_ADDRESS ( (uint8_t *) 0x00030000 )
#define RAM3_SIZE          ( 32 * 1024 )

/**
 * @brief Creates an array of HeapRegion_t definitions, with an index for each of the three RAM regions,
 *        and terminates the array with a HeapRegion_t structure containing a NULL address.
 *        The HeapRegion_t structures must appear in start address order, with the structure
 *        that contains the lowest start address appearing first.
 */
const HeapRegion_t xHeapRegions[] = {
    { RAM1_START_ADDRESS, RAM1_SIZE },
    { RAM2_START_ADDRESS, RAM2_SIZE },
    { RAM3_START_ADDRESS, RAM3_SIZE },
    { NULL, 0 } /* Marks the end of the array. */
};

/**
 * @brief Main function initializes the heap regions and can include additional application code.
 */
int main(void)
{
    /**
     * @brief Initialize the heap regions with the defined addresses and sizes.
     */
    vPortDefineHeapRegions(xHeapRegions);

    /* Add application code here. */

    return 0;
}
```
- Mark the end of the array with a `HeapRegion_t` structure that has its `pucStartAddress` member set to `NULL`.
---
- *The linking phase* of the build process allocates `a RAM address` to each variable.
- The RAM available for use by the *linker* is normally described by a linker configuration file, such as a *linker script*.
- a more convenient and maintainable example. It declares an array called ucHeap.
- *ucHeap* is a normal `variable`, so it becomes part of the data allocated to RAM1 by the `linker`.
- The first HeapRegion_t structure in the xHeapRegions array describes *the start address* and `size of ucHeap`, so `ucHeap` becomes part of the memory managed by heap_5.
```C
/* Define the start address and size of the two RAM regions not used by the linker. */
#define RAM2_START_ADDRESS ((uint8_t *) 0x00020000)
#define RAM2_SIZE (32 * 1024)
#define RAM3_START_ADDRESS ((uint8_t *) 0x00030000)
#define RAM3_SIZE (32 * 1024)

/* Declare an array that will be part of the heap used by heap_5. The array will be placed in RAM1 by the linker. */
#define RAM1_HEAP_SIZE (30 * 1024)

static uint8_t ucHeap[RAM1_HEAP_SIZE];

/* Create an array of HeapRegion_t definitions. Whereas in Listing 3.5 the first entry described all of RAM1,
   so heap_5 will have used all of RAM1, this time the first entry only describes the ucHeap array,
   so heap_5 will only use the part of RAM1 that contains the ucHeap array. The HeapRegion_t structures
   must still appear in start address order, with the structure that contains the lowest start address
   appearing first. */
const HeapRegion_t xHeapRegions[] = {
    {ucHeap, RAM1_HEAP_SIZE},
    {RAM2_START_ADDRESS, RAM2_SIZE},
    {RAM3_START_ADDRESS, RAM3_SIZE},
    {NULL, 0} /* Marks the end of the array. */
};
```
### Heap Related Utility Functions and Macros
#### Defining the Heap Start Address
> Heap_1, heap_2 and heap_4 allocate memory from `a statically allocated array` dimensioned by configTOTAL_HEAP_SIZE.

- Declaring the array in the application code enables the application writer to specify its *start address*.
- If `configAPPLICATION_ALLOCATED_HEAP` is set to 1 in FreeRTOSConfig.h, or left undefined, the application that uses FreeRTOS must allocate `a uint8_t array` called `ucHeap` and dimensioned by the `configTOTAL_HEAP_SIZE` constant.
- The syntax required by the `GCC` compiler to declare the `array` and place the array in a memory section called `.my_heap`.
```C
uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] __attribute__ ( ( section( ".my_heap" ) ) );
```
#### xPortGetFreeHeapSize()
> The `xPortGetFreeHeapSize()` API function returns the number of `free bytes` in the heap at the time the function is called. It does `not` provide information on `heap fragmentation`.
#### xPortGetMinimumEverFreeHeapSize()
> returns the minimum number of unallocated bytes that have ever existed in the heap since the FreeRTOS application started executing.

- The value returned by `xPortGetMinimumEverFreeHeapSize()` indicates `how close` the application has ever come to running out of heap space.
#### Collecting Per-task Heap Usage Statistics
> The `vTaskGetInfo()` API function, documented in section TBD-RB of this book, populates a `TaskStatus_t` structure with information about a task.

If the `configTRACK_TASK_MEMORY_ALLOCATIONS` compile-time constant is set to `1` in FreeRTOSConfig.h, the structure includes the following additional information:
- The number of times the task called `pvPortMalloc().`
- The number of times the task called `vPortFree().`
- The number of heap bytes allocated by the task that have` not yet been freed` by any task at the time `vTaskGetInfo()` was called.
- The `maximum amount of heap memory` allocated by the task at any given time since the task started.
### Malloc Failed Hook Functions
> The malloc failed hook (or callback) is an `application-provided` function that gets called if `pvPortMalloc() `returns `NULL`.

You must set `configUSE_MALLOC_FAILED_HOOK` to `1` in `FreeRTOSConfig.h` in order for the callback to occur.
Production systems should gracefully *recover* from allocation failures.
```C
void vApplicationMallocFailedHook( void );
```
#### Placing Task Stacks in Fast Memory
> FreeRTOS uses the `pvPortMallocStack()` and `vPortFreeStack()` macros to optionally enable `stacks` that are allocated within the FreeRTOS API code to have their `own memory allocator`.

- If you want the stack to come from the heap managed by `pvPortMalloc()` then leave `pvPortMallocStack() `and `vPortFreeStack()` undefined as they default to calling `pvPortMalloc()` and `vPortFree()`, respectively.
### Using Static Memory Allocation
> Static memory allocation allows the developer to `explicity` create every memory block needed by the application.

- All required memory is known at *compile time*.
- All memory is *deterministic*.
---
- The main complication is the addition of *a few additional user functions* to `manage some kernel memory`
- The second complication is the need to ensure all static memory is `declared` in *a suitable scope*.
#### Enabling Static Memory Allocation
> The kernel enables all the s`tatic versions` of `the kernel functions`.

- Static memory allocation is enabled by setting `configSUPPORT_STATIC_ALLOCATION` to `1` in `FreeRTOSConfig.h`.
#### Static Internal Kernel Memory\
> When the static memory allocator is enabled, the `idle task` and the `timer task `(if enabled) will use static memory supplied by user functions.

##### vApplicationGetTimerTaskMemory
If `configSUPPORT_STATIC_ALLOCATION` and `configUSE_TIMERS` are both enabled, the kernel will call `vApplicationGetTimerTaskMemory()` to allow the application to create and `return a memory buffer` for the `timer task TCB` and the `timer task stack`.The function will also return the `size` of the timer task stack.

- Since there is *only* `a single timer task` in any system including SMP, a valid solution to *the timer task memory problem* is to allocate *static buffers* in the `vApplicationGetTimeTaskMemory()` function and return `the buffer pointers` to the kernel.
```c
void vApplicationGetTimerTaskMemory( StaticTask_t **ppxTimerTaskTCBBuffer, StackType_t **ppxTimerTaskStackBuffer, uint32_t *pulTimerTaskStackSize ) {
    /* If the buffers to be provided to the Timer task are declared inside this function then they must be declared static - 
       otherwise they will be allocated on the stack and hence would not exist after this function exits. */
    static StaticTask_t xTimerTaskTCB;
    static StackType_t uxTimerTaskStack[ configMINIMAL_STACK_SIZE ];

    /* Pass out a pointer to the StaticTask_t structure in which the Timer task's state will be stored. */
    *ppxTimerTaskTCBBuffer = &xTimerTaskTCB;

    /* Pass out the array that will be used as the Timer task's stack. */
    *ppxTimerTaskStackBuffer = uxTimerTaskStack;

    /* Pass out the stack size of the array pointed to by *ppxTimerTaskStackBuffer. 
       Note the stack size is a count of StackType_t */
    *pulTimerTaskStackSize = sizeof(uxTimerTaskStack) / sizeof(*uxTimerTaskStack);
}
```

##### vApplicationGetIdleTaskMemory
> The `vApplicationGetIdleTaskMemory function` is called to allow the application to create the needed buffers for **the "main" idle task**.

- The *idle task* performs some housekeeping and can also trigger the `user's vTaskIdleHook()` if it is enabled.
- In a symmetric multiprocessing system (SMP) there are also `non-housekeeping idle tasks` for each of the `remaining cores`, but these are statically allocated internally to `configMINIMUM_STACK_SIZE` bytes.
```C
void vApplicationGetIdleTaskMemory( StaticTask_t **ppxIdleTaskTCBBuffer, StackType_t **ppxIdleTaskStackBuffer, uint32_t *pulIdleTaskStackSize ) {
    static StaticTask_t xIdleTaskTCB;
    static StackType_t uxIdleTaskStack[ configMINIMAL_STACK_SIZE ];

    /* Pass out a pointer to the StaticTask_t structure in which the Idle task's state will be stored. */
    *ppxIdleTaskTCBBuffer = &xIdleTaskTCB;

    /* Pass out the array that will be used as the Idle task's stack. */
    *ppxIdleTaskStackBuffer = uxIdleTaskStack;

    /* Pass out the stack size of the array pointed to by *ppxIdleTaskStackBuffer.
       Note the stack size is a count of StackType_t. */
    *pulIdleTaskStackSize = configMINIMAL_STACK_SIZE;
}
```
## Task Management
### Introduction
- [ ] How FreeRTOS allocates *processing time* to each task in an application.
- [ ] How FreeRTOS chooses which task should *execute* at any given time.
- [ ] How the r*elative priority* of each task affects system behavior.
- [ ] The *states* that a task can exist in.
---
- [ ] How to implement tasks.
- [ ] How to create one or more instances of a task.
- [ ] How to use the task parameter.
- [ ] How to change the priority of a task that has already been created.
- [ ] How to delete a task.
- [ ] How to implement periodic processing using a task. (A later chapter describes how to do the same using software timers.)
- [ ] When the idle task will execute and how it can be used.
### Task Functions
> Each task is a small program in `its own right`. It has an `entry point`, will normally `run forever` in an `infinite loop`, and does `not exit`.
```c
void vATaskFunction( void * pvParameters );
```
- A single task function definition can be used to create any number of tasks where each created task is `a separate execution instance`.
- Each instance has its *own stack* and thus its own copy of any automatic (stack) variables defined within *the task itself*.
### Top Level Task States
> If the processor running the application includes a single core, then only one task may be executing at any given time.

- `The FreeRTOS scheduler` is the *only* entity that can switch a task in and out of the Running state.

### Task Creation
> Each task requires two blocks of RAM: one to hold its Task Control Block (`TCB`) and one to store its `stack`.

| Task Type              | Specific Naming Requirement | Description                                                                                       |
| ---------------------- | --------------------------- | ------------------------------------------------------------------------------------------------- |
| *Static Allocation*    | "Static"                    | Uses `pre-allocated` RAM blocks for TCB and stack;                                                |
| *Dynamic Allocation*   | None                        | Allocates RAM for TCB and stack at runtime from the `system heap`; uses `standard API functions`. |
| *Restricted Execution* | "Restricted"                | `limiting` system memory access.                                                                  |
| *Privileged Execution* | None                        | Tasks created with `standard API functions`, having full access to system memory.                 |
| *SMP Affinity Setting* | "Affinity"                  | Allows specifying the core a task runs on for `multi-core CPUs`; uses "Affinity" named functions. |
#### xTaskCreate()
> `xTaskCreateStatic()` has `two additional parameters` that point to the memory pre-allocated to hold the task's data structure and stack, respectively.
```c
/**
 * @brief Creates a new task and starts its execution.
 *
 * This function creates a new task with the specified properties and starts
 * its execution. The task will be scheduled according to its priority level.
 *
 * @param pvTaskCode A pointer to the task function implementation.
 * @param pcName A human-readable name for the task. It is used only for debugging purposes.
 *               Maximum length is defined by configMAX_TASK_NAME_LEN.
 * @param usStackDepth The number of words allocated for the task's stack.
 *                     For 32-bit architectures, this is effectively the number of 32-bit words.
 * @param pvParameters A void pointer passed to the task function as a parameter.
 * @param uxPriority The priority of the task. Lower numbers indicate higher priorities.
 *                   Valid range is from 0 to (configMAX_PRIORITIES - 1).
 * @param pxCreatedTask A pointer to a variable where the created task's handle will be stored.
 *                      Can be NULL if the handle is not needed.
 *
 * @return Returns pdPASS if the task was created successfully, otherwise returns pdFAIL if there
 *         was insufficient heap memory to create the task.
 */
BaseType_t xTaskCreate( TaskFunction_t pvTaskCode, 
					    const char * const pcName, 
                        configSTACK_DEPTH_TYPE usStackDepth, 
                        void * pvParameters, 
                        UBaseType_t uxPriority, 
                        TaskHandle_t * pxCreatedTask );
```
### Task Priorities
> The FreeRTOS scheduler always ensures `the highest priority` task that can run is the task selected to enter the Running state. Tasks of `equal priority` are transitioned into and out of the Running state in turn.

- The `vTaskPrioritySet()` API function *changes* a task's priority after its creation.
#### Generic Scheduler
> **C code**:It does not impose an upper limit on configMAX_PRIORITEIS.

- In general, it is advisable to *minimize* configMAX_PRIORITIE because more values require more RAM and will result in *a longer worst-case* execution time.
#### Architecture-Optimized Scheduler
> **Assembly code**:The architecture optimized implementation imposes a maximum value for configMAX_PRIORITIES of 32 on 32-bit architectures and 64 on 64-bit architectures.

### Time Measurement and the Tick Interrupt
> The scheduler executes at the end of each time slice to select the next task to run.

- *A periodic interrupt*, called the '`tick interrupt`', is used for this purpose.
- The `configTICK_RATE_HZ` compile-time configuration constant sets the `frequency` of the tick interrupt, and so also the `length` of each time slice.
- The time *between two tick interrupts* is called the '`tick period`'—so one time slice equals ==one tick period==.
---
- FreeRTOS API calls specify time in *multiples of tick periods*, often referred to simply as '`ticks`'.
- The `pdMS_TO_TICKS()` macro converts a time specified in `milliseconds` into a time specified in ticks.
- The resolution available `depends on` the defined tick frequency, and `pdMS_TO_TICKS()` cannot be used if the tick frequency is above 1KHz (if `configTICK_RATE_HZ` is greater than `1000`).
### Expanding the Not Running State
#### The Blocked State
> A task waiting for an `event` is said to be in the 'Blocked' state, a sub-state of the `Not Running state.`

- *Temporal (time-related) events*— these events occur either when `a delay period` expires or `an absolute time` is reached.
- *Synchronization events*— these events originate from another task or interrupt.
#### The Suspended State
> Suspended is also a sub-state of `Not Running.` Tasks in the Suspended state are **not available** to the scheduler.

- The only way to *enter* the Suspended state is through a call to the `vTaskSuspend()` API function.
- The only way *out* is through a call to the `vTaskResume()` or `xTaskResumeFromISR()` API functions.
#### The Ready State
> Tasks that are in the Not Running state and are not Blocked or Suspended are said to be in the Ready state.

- They can `run`, and are therefore '`ready`' to run, but are not currently in the Running state.
#### Completing the State Transition Diagram
#todo Figure 4.7 Full

- *vTaskDelay()*
- The length of time the task remains in the blocked state is specified by the `vTaskDelay()` parameter, but the time at which the task `leaves` the blocked state is relative to the time at which vTaskDelay() was called.
```c
/**
 * @brief Delay a task until a specified time.
 *
 * @details This function places the calling task into the Blocked state until the
 * scheduler next runs, or until the specified time has elapsed. The task will not
 * use any processing time while it is in the Blocked state, and will only use
 * processing time when there is actually work to be done.
 *
 * @param xTicksToDelay The number of tick interrupts to wait before moving the
 * task back to the Ready state. The actual time delay depends on the tick rate
 * set by the system.
 *
 * @note The vTaskDelay() function is only available when INCLUDE_vTaskDelay is
 * set to 1 in FreeRTOSConfig.h.
 *
 * @code
 * void vTaskDelay( TickType_t xTicksToDelay );
 * @endcode
 *
 * @param xTicksToDelay The number of tick interrupts that the calling task will
 * remain in the Blocked state before being transitioned back into the Ready state.
 * For example, if a task called vTaskDelay( 100 ) when the tick count was 10,000,
 * then it would immediately enter the Blocked state, and remain in the Blocked
 * state until the tick count reached 10,100.
 *
 * @note The macro pdMS_TO_TICKS() can be used to convert a time specified in
 * milliseconds into a time specified in ticks. For example, calling
 * vTaskDelay( pdMS_TO_TICKS( 100 ) ) results in the calling task remaining in the
 * Blocked state for 100 milliseconds.
 */
void vTaskDelay( TickType_t xTicksToDelay );
```
- The *idle* task is created automatically when the scheduler is started, to ensure there is always `at least one` task that can `run` (at least one task in the `Ready` state).
- *vTaskDelayUntil()*
- The parameters to `vTaskDelayUntil()` specify, instead, the exact tick count value at which the calling task should be moved from the `Blocked` state into the `Ready` state.
- `vTaskDelayUntil()` is the API function to use when a fixed *execution period* is required (where you want your task to execute periodically with a fixed frequency), as the time at which the calling task is unblocked is absolute.
```c
/**
 * @brief Delay a task until a specified absolute time.
 *
 * @details This function places the calling task into the Blocked state until the
 * specified tick count is reached, at which point the task will be moved back to the
 * Ready state. This is used when a fixed execution period is required, meaning the task
 * should execute periodically with a fixed frequency. The unblocking time is absolute,
 * not relative to when the function was called.
 *
 * @param pxPreviousWakeTime A pointer to a variable that holds the tick count value
 * at which the task last entered the Blocked state. This variable is updated
 * automatically by the vTaskDelayUntil() function and should be initialized to the
 * current tick count before its first use.
 *
 * @param xTimeIncrement The interval in ticks between desired executions of the task.
 * This value is used to calculate the next wake time from the time at which the
 * task last entered the Blocked state.
 *
 * @note The macro pdMS_TO_TICKS() can be used to convert a time specified in
 * milliseconds into a time specified in ticks, which can be used for setting
 * xTimeIncrement.
 *
 * @code
 * void vTaskDelayUntil( TickType_t *pxPreviousWakeTime, TickType_t xTimeIncrement );
 * @endcode
 *
 * @param pxPreviousWakeTime A pointer to a variable that holds the time at which the
 * task last left the Blocked state (was 'woken' up). This time is used as a reference
 * point to calculate the next unblock time.
 *
 * @param xTimeIncrement The period in ticks at which the task should execute. This
 * is an absolute delay time, not a relative delay as with vTaskDelay().
 */
void vTaskDelayUntil( TickType_t *pxPreviousWakeTime, TickType_t xTimeIncrement );
```

### IDIE
#### The Idle Task
> There must always be `at least` one task that can enter the `Running state`. To ensure this is the case, the scheduler automatically creates an `Idle` task when `vTaskStartScheduler()` is called.

- The `idle` task has the lowest possible priority (`priority zero`), to ensure it never prevents `a higher priority` application task from entering the Running state.
- The `configIDLE_SHOULD_YIELD` compile time configuration constant in `FreeRTOSConfig.h` can be used to prevent the Idle task from consuming processing time that would be more productively *allocated to applications* tasks that also have a priority of `0`.
- Running at the `lowest` priority ensures the Idle task is transitioned out of the `Running` state` as soon as` a higher priority task enters the `Ready` state.
- The Idle task is responsible for `cleaning up` *kernel resources* used by tasks that deleted themselves.
#### Idle Task Hook Functions
> It is possible to add `application specific functionality` directly into the i`dle` task through the use of an idle hook (or idle callback) function, which is a function that is called `automatically` by the idle task `once per iteration` of the idle task loop.

- Executing `low priority`, `background`, or `continuous processing` functionality *without the RAM overhead* of creating application tasks for the purpose.
- Measuring the *amount* of `spare processing capacity`.
- Placing the processor into a *low power mode*, providing an `easy` and `automatic` method of saving power whenever there is no application processing to be performed (although the achievable power saving is less than that achieved by tick-less idle mode).
#### Limitations on the Implementation of Idle Task Hook Functions
- An Idle task hook function must never attempt to *block* or *suspend* `itself`.
- If an application task uses the `vTaskDelete()` API function to delete itself, then the Idle task hook must always *return* to its caller within a reasonable time period.
- `configUSE_IDLE_HOOK` must be set to `1` in `FreeRTOSConfig.h` for the idle hook function to get called.
- Idle task hook functions must have the name and prototype
```C
void vApplicationIdleHook( void );
```
### Changing the Priority of a Task
#### vTaskPrioritySet()
> The `vTaskPrioritySet()` API function changes the priority of a task `after `the scheduler has been *started*.

- Using the `vTaskPrioritySet()` API function to change the priority of `two tasks` *relative to each other*.
- Each task can both query and set its own priority by using `NULL` in place of a valid task handle. A task handle is `only required` when a task wishes to `reference` a task `other` than itself
### Deleting a Task
#### vTaskDelete()
> If a task that was created using `dynamic memory allocation` later deletes itself, the `Idle` task is responsible for freeing the memory allocated for use, such as the deleted `task's data structure` and `stack`.

- Only memory allocated to a task by the `kernel` itself is freed automatically when the task is deleted.
```C
```c
/**
 * @brief Deletes a task.
 *
 * This function deletes a task, which can be either the calling task or any other task
 * specified by a task handle. If the task being deleted is not the calling task, the
 * task handle must be a valid handle to the task. A task can delete itself by passing
 * NULL as the task handle.
 *
 * @param xTaskToDelete A handle to the task that is to be deleted. This can be:
 *   - The handle returned by xTaskCreate() when the task was created.
 *   - The handle returned by xTaskCreateStatic() when the task was created using
 *     static allocation.
 *   - NULL, in which case the calling task (the task that calls this function) will
 *     be deleted.
 *
 * Example usage:
 * @code{c}
 * TaskHandle_t myTaskHandle = NULL;
 * xTaskCreate(myTaskFunction, "My Task", 1000, NULL, 1, &myTaskHandle);
 * // ... Task is running ...
 * vTaskDelete(myTaskHandle); // Delete the task using its handle
 * @endcode
 */
void vTaskDelete(TaskHandle_t xTaskToDelete);
```
### Thread Local Storage and Reentrancy
> `Thread Local Storage` allows an application developer to store `arbitrary data` in the `Task Control Block` of each task.

- This feature is most commonly used to store data which would normally be stored in a `global variable` by **non-reentrant functions**.
- *A reentrant function* is a function which can safely run from *multiple threads* without any `side effects`.
#### Application Thread Local Storage
> Application developers may also define a set of application `specific pointers` to be included in the `task control block`.

- The `vTaskSetThreadLocalStoragePointer` and `pvTaskGetThreadLocalStoragePointer` functions may be used respectively to set and get the value of each thread local storage pointer at `runtime`.
### Scheduling Algorithms
#### A Recap of Task States and Events
- Tasks can wait in the *Blocked* state for an event and they are *automatically* moved back to the *Ready* state when the event occurs.
#### Selecting the Scheduling Algorithm
> The scheduling algorithm is `the software routine` that decides which `Ready` state task to transition into the `Running` state.

- `A Round Robin scheduling algorithm` does not guarantee time is shared equally between tasks of `equal priority,` only that *Ready* state tasks of equal priority enter the *Running* state in turn.
#### Prioritized Preemptive Scheduling with Time Slicing
- *Fixed Priority*:Scheduling algorithms described as 'Fixed Priority' do `not change` the priority assigned to the tasks `being scheduled`, but also do `not prevent` the tasks themselves from `changing` their own priority or that of other tasks.
- *Preemptive scheduling algorithms* will immediately 'preempt' the Running state task if a task that has `a priority higher` than the `Running` state task enters the `Ready `state.
- *A time slice* is equal to the time between two RTOS `tick interrupts`.
- If `configIDLE_SHOULD_YIELD` is set to `1` then the *Idle* task *yields* (voluntarily gives up whatever remains of its allocated time slice) on each iteration of its loop if there are *other Idle priority tasks* in the `Ready` state.The task selected to enter the `Running `state after the Idle task does not execute for `an entire time slice`, but instead executes for whatever remains of the time slice *during which the Idle task yielded*.
#### Prioritized Preemptive Scheduling without Time Slicing
> If time slicing is not used, then the scheduler only selects a new task to enter the Running state when either:

- `A higher priority` task enters the `Ready` state.
- The task in the `Running` state enters the `Blocked` or `Suspended` state.
### Cooperative Scheduling
- When using the cooperative scheduler (and therefore assuming application-provided interrupt service routines do not explicitly request *context switches*)
- A context switch only occurs when the `Running` state task enters the `Blocked` state, or the `Running` state task explicitly yields (manually requests a re-schedule) by calling *taskYIELD()*.
- Tasks are never `preempted`, so `time slicing` cannot be used.
- Using the cooperative scheduler normally makes it easier to `avoid problems` caused by *simultaneous access* than when using the preemptive scheduler.
- Using the cooperative scheduler makes systems *less responsive* than when using the preemptive scheduler
## Queue Management
### Introduction
> `Queues` provide a `task-to-task`, `task-to-interrupt`, and `interrupt-to-task` communication mechanism.

- [ ] How to `create` a queue.
- [ ] How a queue `manages the data` it contains.
- [ ] How to `send` data to a queue.
- [ ] How to `receive` data from a queue.
- [ ] What it means to `block` on a queue.
- [ ] How to block on `multiple queues`.
- [ ] How to `overwrite` data in a queue.
- [ ] How to `clear` a queue.
- [ ] The effect of `task priorities` when `writing` to and `reading` from a queue.
### Characteristics of a Queue
#### Data Storage
> A queue can hold a `finite number` of fixed size data items
- It is also possible to write to the front of a queue, and to overwrite data that is already at the front of a queue.

| Queuing by *copy*      | The data sent to the queue is `copied byte` for byte into the queue.                |
| ---------------------- | ----------------------------------------------------------------------------------- |
| Queuing by *reference* | The queue only holds `pointers` to the data sent to the queue, not the data itself. |
- Queuing by copy allows data to pass across `memory protection boundaries`.
#### Access by Multiple Tasks
> Queues are objects in their own right and can be accessed by any task or ISR that knows of their existence.

#### Blocking on Queue Reads
- A task that is in the `Blocked` state waiting for data to become `available` from a queue is *automatically moved* to the `Ready` state when `another task` or `interrupt` places data into the queue.
- The task will also be *moved automatically* from the `Blocked` state to the `Ready` state if `the specified block time` expires before data becomes available.
- Queues can have `multiple readers`, so it is possible for a `single queue` to have more than one task `blocked` on it waiting for data.
- The task that is `unblocked` is always the *highest priority* task that is waiting for data.
- If two or more blocked tasks have `equal priority`, then the task that is unblocked is the one that has been *waiting the longest*.
#### Blocking on Queue Writes
- Queues can have `multiple writers`, so it is possible for a full queue to have more than one task blocked on it waiting to complete a send operation.
#### Blocking on Multiple Queues
- Queues can be grouped into `sets`, allowing a task to enter the `Blocked` state to wait for data to become available on any of the queues in the set.
#### Creating Queues: Statically Allocated and Dynamically Allocated Queues
> Each queue requires `two blocks` of `RAM`, the first to hold its `data structure`, and the second to hold queued `data`.
- `xQueueCreate()` allocates the required RAM from the *heap* (dynamically).
- `xQueueCreateStatic()` uses *pre-allocated* RAM passed into the function as *parameters*.
### Using a Queue
#### xQueueCreate()
```c
/**
 * @brief Creates a new queue and returns a handle to the queue.
 * 
 * This function creates a new queue capable of holding a specified number of
 * items, where each item has a specified size.
 * 
 * @param uxQueueLength The maximum number of items that the queue being
 * created can hold at any one time.
 * @param uxItemSize The size in bytes of each data item that can be
 * stored in the queue.
 * 
 * @return QueueHandle_t
 * @retval NULL If the queue cannot be created because there is insufficient
 * heap memory available for FreeRTOS to allocate the queue data structures
 * and storage area. More information on the FreeRTOS heap can be found in
 * Chapter 2.
 * @retval Non-NULL If the queue was created successfully, and the returned
 * value is the handle to the created queue.
 * 
 * @note Listing 5.1 shows the prototype for this function.
 * @see xQueueCreateStatic() for a version that uses pre-allocated memory.
 */
QueueHandle_t xQueueCreate(
    UBaseType_t uxQueueLength,
    UBaseType_t uxItemSize
);
```
#### xQueueSendToBack() and xQueueSendToFront()
> `xQueueSendToBack()` sends data to the `back` (tail) of a queue.
```c
/**
 * @brief Sends an item to the back of a queue.
 * 
 * This function places the specified item at the back of the queue. It is
 * equivalent to xQueueSend(), and behaves the same way as xQueueSendToBack().
 * If the queue is full, then this function will block for a specified amount of
 * time until space becomes available, or until a timeout occurs.
 * 
 * @param xQueue The handle to the queue to which the item is being sent.
 * @param pvItemToQueue A pointer to the item that is to be sent.
 * @param xTicksToWait The maximum amount of time in ticks to wait for space
 * to become available on the queue if the queue is full.
 * 
 * @return BaseType_t
 * @retval pdPASS If the item was successfully sent to the queue.
 * @retval errQUEUE_FULL If the queue was full and the item could not be sent
 * within the specified time.
 * 
 * @note Never call this function from an interrupt service routine; use
 * xQueueSendToBackFromISR() instead. See Chapter 7 for more details.
 * @see xQueueSendToFront() for sending to the front of the queue.
 */
BaseType_t xQueueSendToBack(
    QueueHandle_t xQueue,
    const void * pvItemToQueue,
    TickType_t xTicksToWait
);
```

> `xQueueSendToFront()` sends data to the `front` (head) of a queue.

```c
/**
 * @brief Sends an item to the front of a queue.
 * 
 * This function places the specified item at the front of the queue. If the
 * queue is full, then this function will block for a specified amount of time
 * until space becomes available, or until a timeout occurs.
 * 
 * @param xQueue The handle to the queue to which the item is being sent.
 * @param pvItemToQueue A pointer to the item that is to be sent.
 * @param xTicksToWait The maximum amount of time in ticks to wait for space
 * to become available on the queue if the queue is full.
 * 
 * @return BaseType_t
 * @retval pdPASS If the item was successfully sent to the queue.
 * @retval errQUEUE_FULL If the queue was full and the item could not be sent
 * within the specified time.
 * 
 * @note Never call this function from an interrupt service routine; use
 * xQueueSendToFrontFromISR() instead. See Chapter 7 for more details.
 * @see xQueueSendToBack() for sending to the back of the queue.
 */
BaseType_t xQueueSendToFront(
    QueueHandle_t xQueue,
    const void * pvItemToQueue,
    TickType_t xTicksToWait
);
```
#### xQueueReceive()
```c
/**
 * @brief Receives an item from a queue.
 *
 * This function receives an item from the front of the specified queue. If the
 * queue is empty, then this function will block for a specified amount of time
 * until data becomes available, or until a timeout occurs.
 *
 * @param xQueue The handle to the queue from which the item is to be received.
 * @param pvBuffer A pointer to the buffer into which the received item will be
 * copied. The buffer must be large enough to hold the data.
 * @param xTicksToWait The maximum amount of time in ticks to wait for data to
 * become available on the queue if the queue is empty.
 *
 * @return BaseType_t
 * @retval pdPASS If the item was successfully received from the queue.
 * @retval errQUEUE_EMPTY If the queue was empty and no data could be received
 * within the specified time.
 *
 * @note The block time is specified in tick periods, and the absolute time it
 * represents is dependent on the tick frequency. The macro pdMS_TO_TICKS() can
 * be used to convert a time specified in milliseconds into a time specified in
 * ticks. Setting xTicksToWait to portMAX_DELAY will cause the task to wait
 * indefinitely, provided INCLUDE_vTaskSuspend is set to 1 in FreeRTOSConfig.h.
 */
BaseType_t xQueueReceive(
    QueueHandle_t xQueue,
    void * const pvBuffer,
    TickType_t xTicksToWait
);
```
#### uxQueueMessagesWaiting()
> `uxQueueMessagesWaiting()` queries the `number` of items currently in a queue.
```C
/**
 * @brief Queries the number of items currently in a queue.
 *
 * This function returns the number of items that are currently in the
 * specified queue. It can be used to determine if the queue is empty or full.
 *
 * @param xQueue The handle to the queue being queried.
 *
 * @return UBaseType_t The number of items currently in the queue.
 * If zero is returned, then the queue is empty.
 *
 * @note Never call this function from an interrupt service routine; use
 * uxQueueMessagesWaitingFromISR() instead.
 */
UBaseType_t uxQueueMessagesWaiting(
    QueueHandle_t xQueue
);
```

- The tasks that send to the queue have a `lower priority` than the task that receives from the queue. This means the queue should never contain more than `one` item because, as soon as data is sent to the queue the receiving task will unblock, `pre-empt` the sending task (because it has a higher priority), and `remove` the data, leaving the queue *empty* once again.
### Receiving Data From Multiple Sources
> It is common in FreeRTOS designs for a task to receive data from more than one source. The receiving task needs to know where the data came from to determine what to do with it.

### Working with Large or Variable Sized Data
#### Queuing Pointers
> Transferring `pointers` is morefficient in both `processing time` and the `amount` of RAM required to create the queue.

- The owner of the RAM being pointed to is clearly *defined*.
- The RAM being pointed to remains *valid*.
#### Using a Queue to Send Different Types and Lengths of Data
> FreeRTOS `message buffers` are a `lighter weight alternative` to queues that hold **variable length** data.

### Using a Queue to Create a Mailbox
> The term mailbox is used to refer to a queue that has a length of one.

- A mailbox is used to hold data that can be read by any task, or any interrupt service routine.
- The sender *overwrites* the value in the mailbox. The receiver reads the value from the mailbox, but does not remove the value from the mailbox.
#### xQueueOverwrite()
> xQueueOverwrite() must only be used with queues that have a length of `one`.

#### xQueuePeek()
> xQueuePeek() receives (reads) an item from a queue `without removing` the item from the queue.

```C
BaseType_t vReadMailbox( Example_t *pxData ) {
    TickType_t xPreviousTimeStamp;
    BaseType_t xDataUpdated;

    /* This function updates an Example_t structure with the latest value received from the mailbox.
      Record the time stamp already contained in *pxData before it gets overwritten by the new data. */
    xPreviousTimeStamp = pxData->xTimeStamp;

    /* Update the Example_t structure pointed to by pxData with the data contained in the mailbox.
       If xQueueReceive() was used here then the mailbox would be left empty, and the data could not then be read by any other tasks.
       Using xQueuePeek() instead of xQueueReceive() ensures the data remains in the mailbox.
       A block time is specified, so the calling task will be placed in the Blocked state to wait for the mailbox to contain data should the mailbox be empty.
       An infinite block time is used, so it is not necessary to check the value returned from xQueuePeek(),
       as xQueuePeek() will only return when data is available. */
    xQueuePeek( xMailbox, pxData, portMAX_DELAY );

    /* Return pdTRUE if the value read from the mailbox has been updated since this function was last called.
       Otherwise return pdFALSE. */
    if( pxData->xTimeStamp > xPreviousTimeStamp ) {
        xDataUpdated = pdTRUE;
    } else {
        xDataUpdated = pdFALSE;
    }
    return xDataUpdated;
}
```
## Software Timer Management
### Introduction and Scope
> Software timers are used to `schedule` the execution of a function at `a set time in the future`, or `periodically` with a fixed frequency.

- Software timers are implemented by, and are under the control of, the FreeRTOS kernel. They do not require hardware support, and are not related to `hardware timers `or `hardware counters`.
- software timers do not use any *processing time* unless a software timer `callback function` is actually executing.

*configTIMER_TASK_STACK_DEPTH*
- Sets the size of the stack (in *words*, not bytes) allocated to the timer service task.
#### Scope
- [ ] The characteristics of a software timer compared to the characteristics of a task.
- [ ] The RTOS daemon task. The timer command queue.
- [ ] The difference between a one shot software timer and a periodic software timer.
- [ ] How to create, start, reset and change the period of a software timer.
### Software Timer Callback Functions
> The only thing special about them is their prototype, which must return `void`, and take `a handle to a software timer` as its only parameter.

- They should be kept *short*, and must not enter the `Blocked state`.
- Software timer callback functions execute in the context of `a task that is created automatically` when the FreeRTOS scheduler is started.
### Attributes and States of a Software Timer
#### One-shot and Auto-reload Timers
- Once started, a one-shot timer will execute its callback function *once only*. A one-shot timer can be `restarted manually`, but will not restart itself.
- Once started, an auto-reload timer will *re-start* itself each time it expires, resulting in *periodic* execution of its callback function.
#### Software Timer States
- **Dormant**: A Dormant software timer exists, and can be referenced by its handle, but is not running, so its callback functions will not execute.
- **Running**: A Running software timer will execute its callback function after a time equal to its period has elapsed since the software timer entered the Running state, or since the software timer was last reset.
#### xTimerDelete()
- The xTimerDelete() API function deletes a timer. A timer can be deleted at *any time*.
```c
/**
 * Delete a timer.
 *
 * @param xTimer The handle of the timer being deleted.
 *               This is the handle returned by xTimerCreate();
 *               the timer being deleted must have been created using xTimerCreate().
 *
 * @param xTicksToWait Specifies the time, in ticks, that the calling task should
 *                      be held in the Blocked state to wait for the delete command
 *                      to be successfully sent to the timer command queue,
 *                      should the queue already be full when xTimerDelete() was called.
 *                      xTicksToWait is ignored if xTimerDelete() is called before
 *                      the scheduler is started.
 *
 * @return pdPASS pdPASS will be returned if the command was successfully sent
 *                 to the timer command queue.
 * @return pdFAIL pdFAIL will be returned if the delete command could not be sent
 *                to the timer command queue even after xTicksToWait ticks had passed.
 */
BaseType_t xTimerDelete( TimerHandle_t xTimer, TickType_t xTicksToWait );
```
### The Context of a Software Timer
#### The RTOS Daemon (Timer Service) Task
> All software timer `callback functions` execute in the `context` of the same RTOS daemon (or 'timer service') task.

- The daemon task is a standard FreeRTOS task that is created *automatically* when the `scheduler` is started.
- Its priority and stack size are set by the `configTIMER_TASK_PRIORITY` and `configTIMER_TASK_STACK_DEPTH` compile time configuration constants respectively.
- Software timer callback functions must not call FreeRTOS API functions that will result in the calling task entering the *Blocked state*.

### The Timer Command Queue
> Software timer API functions **send commands** from the calling task to the daemon task on a queue called the `timer command queue`.

- The timer command queue is a standard FreeRTOS queue that is created *automatically* when the `scheduler` is started.
- `configTIMER_QUEUE_LENGTH`
### Daemon Task Scheduling
> The daemon task is `scheduled` like any other FreeRTOS task; it will only process commands, or execute timer callback functions, when it is the highest priority task that is able to run.

- The time at which the software timer *being started* will expire is calculated from the time the 'start a timer' command was sent to the `timer command queue`—it is not calculated from the time the daemon task `received`the 'start a timer' command from the timer command queue.
- Commands sent to the timer command queue contain a **time stamp**. The time stamp is used to `account` for any time that passes between `a command being sent` by an application task, and the same command `being processed` by the daemon task.
### Creating and Starting a Software Timer
#### xTimerCreate()
> Software timers are created in the `Dormant` state.
- Software timers can be created `before` the *scheduler* is running, or from a task `after` the scheduler has been started.
```c
/**
 * xTimerCreate - Create a new software timer.
 *
 * @param pcTimerName: A descriptive name for the timer. This is not used by
 * FreeRTOS in any way. It is included purely as a debugging aid.
 * Identifying a timer by a human readable name is much simpler than
 * attempting to identify it by its handle.
 *
 * @param xTimerPeriodInTicks: The timer's period specified in ticks. The
 * pdMS_TO_TICKS() macro can be used to convert a time specified in
 * milliseconds into a time specified in ticks. Cannot be 0.
 *
 * @param xAutoReload: Set xAutoReload to pdTRUE to create an auto-reload
 * timer. Set xAutoReload to pdFALSE to create a one-shot timer.
 *
 * @param pvTimerID: Each software timer has an ID value. The ID is a void
 * pointer, and can be used by the application writer for any purpose. The
 * ID is particularly useful when the same callback function is used by more
 * than one software timer, as it can be used to provide timer specific
 * storage. Use of a timer's ID is demonstrated in an example in this chapter.
 *
 * @param pxCallbackFunction: Software timer callback functions are simply C
 * functions that conform to the prototype shown in Listing 6.1. The
 * pxCallbackFunction parameter is a pointer to the function (in effect,
 * just the function name) to use as the callback function for the software
 * timer being created.
 *
 * @return: If NULL is returned, then the software timer cannot be created
 * because there is insufficient heap memory available for FreeRTOS to
 * allocate the necessary data structure. If a non-NULL value is returned it
 * indicates that the software timer has been created successfully. The
 * returned value is the handle of the created timer. Chapter 3 provides more
 * information on heap memory management.
 *
 * This function creates a new software timer and returns a handle by which
 * the timer can be referenced.
 */
TimerHandle_t xTimerCreate( const char * const pcTimerName,
                            const TickType_t xTimerPeriodInTicks,
                            const BaseType_t xAutoReload,
                            void * const pvTimerID,
                            TimerCallbackFunction_t pxCallbackFunction );
```
#### xTimerStart()
> xTimerStart() is used to start a software timer that is in the `Dormant` state, or `reset` (re-start) a software timer that is in the Running state.

- xTimerStart() can be called `before` the **scheduler** is started, but when this is done, the software timer will not actually start until the time at which the `scheduler starts`.
### The Timer ID
> Each software timer has an ID, which is a tag value that can be used by the application writer for any purpose.

- The ID is stored in a void pointer (`void *`), so it can store an integer value directly, point to any other object, or be used as a function pointer.
- The same callback function can be assigned to *more than one* software timer. When that is done, the callback function parameter is used to determine which software timer expired.
### Changing the Period of a Timer
#### xTimerChangePeriod()
If xTimerChangePeriod() is used to change the period of a timer that is already `running`, then the timer will use the new period value to *recalculate* its expiry time. The recalculated expiry time is relative to when `xTimerChangePeriod()` was called, not relative to when the timer was originally started.

If xTimerChangePeriod() is used to change the period of a timer that is in the `Dormant` state (a timer that is not running), then the timer will calculate an expiry time, and transition to the Running state (the timer will start `running`).
### Resetting a Software Timer
Resetting a software timer means to re-start the timer; the timer's expiry time is `recalculated` to be relative to when *the timer was reset*, rather than when the timer was originally started.
#### xTimerReset()
xTimerReset() can also be used to start a timer that is in the `Dormant` state.
## Interrupt Management
### Introduction
> `Tasks` will only run when there are no `ISRs` running, so the `lowest priority interrupt` will interrupt the `highest priority task`, and there is no way for a task to preempt an ISR.

- [ ] Which FreeRTOS API functions can be used from within an interrupt service routine.
- [ ] Methods of deferring interrupt processing to a task.
- [ ] How to create and use binary semaphores and counting semaphores.
- [ ] The differences between binary and counting semaphores.
- [ ] How to use a queue to pass data into and out of an interrupt service routine.
- [ ] The interrupt nesting model available with some FreeRTOS ports.
### Using the FreeRTOS API from an ISR
#### The **xHigherPriorityTaskWoken** Parameter
> A switch to a higher priority task will `not occur automatically` inside an interrupt.

- Instead, a variable is set to inform the application writer that a *context switch* should be performed.
- Interrupt safe API functions (those that end in "FromISR") have a pointer parameter called `pxHigherPriorityTaskWoken` that is used for this purpose.
- If a context switch should be performed, then the interrupt safe API function will set `*pxHigherPriorityTaskWoken` to `pdTRUE`.
- This will ensure that the interrupt returns **directly** to `the highest priority` **Ready state task**.
#### portYIELD_FROM_ISR()、portEND_SWITCHING_ISR()
- `taskYIELD()` is a macro that can be called in a task to request a `context switch`.
- `portYIELD_FROM_ISR()` and `portEND_SWITCHING_ISR()` are both interrupt safe versions of taskYIELD().
### Deferred Interrupt Processing
> It is normally considered best practice to keep ISRs as short as possible.
- ISRs can *disrupt* (add 'jitter' to) both the start time, and the execution time, of a task.
- Depending on the architecture on which FreeRTOS is running, it might not be possible to accept any `new interrupts`, or at least `a subset of new interrupts`, while an ISR is executing.
- The application writer needs to consider the consequences of, and guard against, resources such as variables, peripherals, and memory buffers `being accessed` by a task and an ISR at the same time.

> An interrupt service routine must record the `cause` of the interrupt, and `clear` the interrupt.
- The processing necessitated by the interrupt is not trivial.
- It is convenient for the interrupt processing to perform an action that cannot be performed inside an ISR, such as write to a console, or allocate memory.
- The interrupt processing is *not deterministic*—meaning it is not known in advance how long the processing will take.
### Binary Semaphores Used for Synchronization
The interrupt safe version of the Binary Semaphore API can be used to `unblock` a task each time `a particular interrupt` occurs, effectively *synchronizing* the task with the interrupt.

- The binary semaphore can be considered conceptually as **a queue** with `a length of one`. The queue can contain a maximum of one item at any time, so is always either empty or full (hence, binary).
- It often causes confusion as it does `not` follow the same rules as other semaphore usage scenarios, where a task that *takes* a semaphore must always *give* it back
#### xSemaphoreGiveFromISR()
```C
 /**
 * @brief Give a semaphore from an ISR.
 *
 * This function is used to give a semaphore from an Interrupt Service Routine (ISR).
 * If the semaphore is successfully given, and the calling ISR has a higher priority
 * than the task that was unblocked, then *pxHigherPriorityTaskWoken is set to pdTRUE,
 * indicating that a `context switch` should be performed `before` exiting the ISR.
 *
 * @param xSemaphore The semaphore to give. This is a pointer to a SemaphoreHandle_t
 *                   that must have been created using a call to xSemaphoreCreateBinary()
 *                   or a similar function.
 * @param pxHigherPriorityTaskWoken A pointer to a variable that will be set to pdTRUE
 *                                  if giving the semaphore caused a task to unblock,
 *                                  and the unblocked task has a higher priority than
 *                                  the currently running task. If no tasks were
 *                                  blocked on the semaphore, or the unblocked task
 *                                  has a lower or equal priority, then the variable
 *                                  will be set to pdFALSE.
 * @return pdPASS if the semaphore was successfully given, otherwise pdFAIL.
 */
BaseType_t xSemaphoreGiveFromISR( SemaphoreHandle_t xSemaphore,
                                  BaseType_t * const pxHigherPriorityTaskWoken );
```

The `xHigherPriorityTaskWoken` is set to *pdFALSE* before calling `xSemaphoreGiveFromISR()`, then used as the parameter when `portYIELD_FROM_ISR()` is called. A **context switch** will be requested inside the `portYIELD_FROM_ISR()` macro if xHigherPriorityTaskWoken equals *pdTRUE*.
```c
static uint32_t ulExampleInterruptHandler( void )
{
    BaseType_t xHigherPriorityTaskWoken; /* The xHigherPriorityTaskWoken parameter must be initialized to pdFALSE as it will get set to pdTRUE inside the interrupt safe API function if a context switch is required. */

    xHigherPriorityTaskWoken = pdFALSE;

    /* 'Give' the semaphore to unblock the task, passing in the address of xHigherPriorityTaskWoken as the interrupt safe API function's pxHigherPriorityTaskWoken parameter. */
    xSemaphoreGiveFromISR( xBinarySemaphore, &xHigherPriorityTaskWoken );

    /* Pass the xHigherPriorityTaskWoken value into portYIELD_FROM_ISR().
       If xHigherPriorityTaskWoken was set to pdTRUE inside xSemaphoreGiveFromISR()
       then calling portYIELD_FROM_ISR() will request a context switch.
       If xHigherPriorityTaskWoken is still pdFALSE then calling portYIELD_FROM_ISR()
       will have no effect.
       Unlike most FreeRTOS ports, the Windows port requires the ISR to return a value
       the return statement is inside the Windows version of portYIELD_FROM_ISR(). */
    portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
```

The recommended structure of *a deferred interrupt processing task*, using `a UART receive handler` as an example.
- If the task is waiting without a timeout, it will not know about `the error state`, and will wait forever.
- If the task is waiting with a timeout, then `xSemaphoreTake()` will return *pdFAIL* when the timeout expires, and the task can then `detect and clear the error` the next time it executes.
```c
static void vUARTReceiveHandlerTask( void *pvParameters )
{
    /* xMaxExpectedBlockTime holds the maximum time expected between two interrupts. */
    const TickType_t xMaxExpectedBlockTime = pdMS_TO_TICKS( 500 );

    /* As per most tasks, this task is implemented within an infinite loop. */
    for( ;; )
    {
        /* The semaphore is 'given' by the UART's receive (Rx) interrupt.
           Wait a maximum of xMaxExpectedBlockTime ticks for the next interrupt. */
        if( xSemaphoreTake( xBinarySemaphore, xMaxExpectedBlockTime ) == pdPASS )
        {
            /* The semaphore was obtained. Process ALL pending Rx events before
               calling xSemaphoreTake() again. Each Rx event will have placed
               a character in the UART's receive FIFO, and UART_RxCount() is
               assumed to return the number of characters in the FIFO. */
            while( UART_RxCount() > 0 )
            {
                /* UART_ProcessNextRxEvent() is assumed to process one Rx character,
                   reducing the number of characters in the FIFO by 1. */
                UART_ProcessNextRxEvent();
            }

            /* No more Rx events are pending (there are no more characters in the FIFO),
               so loop back and call xSemaphoreTake() to wait for the next interrupt.
               Any interrupts occurring between this point in the code and the call
               to xSemaphoreTake() will be latched in the semaphore, so will not be lost. */
        }
        else
        {
            /* An event was not received within the expected time.
               Check for, and if necessary clear, any error conditions in the UART
               that might be preventing the UART from generating any more interrupts. */
            UART_ClearErrors();
        }
    }
}
```
### Counting Semaphores
> Counting semaphores can be thought of as **queues** that have a length of `more than one`.

- Tasks are not interested in the data that is stored in the queue—just the **number** of items in the queue.
- `configUSE_COUNTING_SEMAPHORES` must be set to `1` in `FreeRTOSConfig.h` for counting semaphores to be available.

Counting semaphores are typically used for two things:
- *Counting events*:The count value is the difference between the number of events that `have occurred` and the number that `have been processed`.
- *Resource management*:The count value indicates the number of `resources available`.
#### xSemaphoreCreateCounting()
```c
/**
 * @brief Creates a counting semaphore.
 * 
 * Creates a new counting semaphore instance. A counting semaphore is used to 
 * manage access to a set number of resources where the maximum number of 
 * resources and the initial number of resources can be defined.
 * 
 * @param uxMaxCount The maximum value to which the semaphore will count.
 *                   This is the maximum number of resources or events that
 *                   can be managed by the semaphore.
 * @param uxInitialCount The initial count value of the semaphore after it has
 *                       been created. This should be set to zero when counting
 *                       events, and to the total number of resources when
 *                       managing resources.
 * 
 * @return SemaphoreHandle_t 
 * - NULL if the semaphore cannot be created due to insufficient heap memory.
 * - Non-NULL if the semaphore has been created successfully. The returned value
 *   should be stored as the handle to the created semaphore.
 * 
 * @note Chapter 3 provides more information on heap memory management.
 */
SemaphoreHandle_t xSemaphoreCreateCounting( UBaseType_t uxMaxCount, UBaseType_t uxInitialCount );
```
### Deferring Work to the RTOS Daemon Task
> It is also possible to use the `xTimerPendFunctionCallFromISR()` API function to `defer interrupt processing` to the **RTOS daemon task**, which removes the need to `create a separate task for each interrupt`.

- Deferring interrupt processing to the daemon task is called `centralized deferred interrupt processing`.
- `The daemon task` was originally called *the timer service task* because it was originally only used to execute `software timer callback` functions.

Advantages of centralized deferred interrupt processing include:
- *Lower resource usage*
- *Simplified user model*:The deferred interrupt handling function is a standard C function.
Disadvantages of centralized deferred interrupt processing include:
- *Less flexibility*:It is not possible to set the priority of each deferred interrupt handling task separately.
- *Less determinism*:`Commands that were already` in the timer command queue will be processed by the daemon task `before` the 'execute function' command sent to the queue by xTimerPendFunctionCallFromISR().

#### xTimerPendFunctionCallFromISR()
`xTimerPendFunctionCallFromISR()` is the interrupt safe version of `xTimerPendFunctionCall()`.
```c
/**
 * @brief The interrupt safe version of xTimerPendFunctionCall().
 *
 * This function allows a function provided by the application writer to be
 * executed by, and therefore in the context of, the RTOS daemon task. Both
 * the function to be executed and the value of the function's input parameters
 * are sent to the daemon task on the timer command queue. When the function
 * actually executes is therefore dependent on the priority of the daemon task
 * relative to other tasks in the application.
 *
 * @param xFunctionToPend A pointer to the function that will be executed in
 *                        the daemon task. The prototype of the function must be the same.
 * @param pvParameter1 The value that will be passed into the function that
 *                     is executed by the daemon task as that function's
 *                     pvParameter1 parameter. The parameter has a void* type
 *                     to allow it to be used to pass any data type.
 * @param ulParameter2 The value that will be passed into the function that
 *                     is executed by the daemon task as that function's
 *                     ulParameter2 parameter.
 * @param pxHigherPriorityTaskWoken If xTimerPendFunctionCallFromISR() writes
 *                                 to the timer command queue and the RTOS daemon
 *                                 task was in the Blocked state, this function
 *                                 will set *pxHigherPriorityTaskWoken to pdTRUE.
 *                                 If this value is set to pdTRUE, a context switch
 *                                 must be performed before the interrupt is exited.
 *
 * @return BaseType_t
 * - pdPASS: The 'execute function' command was written to the timer command queue.
 * - pdFAIL: The 'execute function' command could not be written to the timer
 *           command queue because the timer command queue was already full.
 * 
 * @note Chapter 6 describes how to set the length of the timer command queue.
 */
BaseType_t xTimerPendFunctionCallFromISR(
    PendedFunction_t xFunctionToPend,
    void *pvParameter1,
    uint32_t ulParameter2,
    BaseType_t *pxHigherPriorityTaskWoken
);
```

The `xTimerPendFunctionCallFromISR()` API function prototype:
```c
void vPendableFunction( void *pvParameter1, uint32_t ulParameter2 );
```
### Using Queues within an Interrupt Service Routine
> `Binary and counting semaphores` are used to communicate events. `Queues` are used to communicate **events** and to transfer **data**.

#### Considerations When Using a Queue From an ISR
> It is `not efficient` to use a queue if data is arriving at a high frequency.

More efficient techniques, that are suitable for production code, include:
- **DMA**:Using `Direct Memory Access` hardware to receive and buffer characters.A *direct to task notification* can then be used to `unblock` the task that will process the buffer only after a break in transmission has been detected.
- **Stream Buffer**:`Copying` each received character into a `thread safe RAM buffer`,Again, a *direct to task notification* can be used to unblock the task that will process the buffer after a complete message has been received, or after a break in transmission has been detected.
### Interrupt Nesting
The priority assigned to a task is in no way related to the priority assigned to an interrupt. Hardware decides when an `ISR` will execute, whereas software decides when a task will execute. An ISR executed in response to a hardware interrupt will `interrupt a task`, but a task *cannot* `pre-empt` an ISR.

> Constants that control interrupt nesting:
- `configMAX_SYSCALL_INTERRUPT_PRIORITY` or `configMAX_API_CALL_INTERRUPT_PRIORITY`Sets the *highest* interrupt priority from which interrupt-safe FreeRTOS API functions can be called.
- `configKERNEL_INTERRUPT_PRIORITY`Sets the interrupt priority used by the **tick interrupt**, and must always be set to the *lowest* possible interrupt priority.

> Each interrupt source has a *`numeric priority`*, and a *`logical priority`*:
- *Numeric priority*:The numeric priority is simply the number assigned to the interrupt priority.
- *Logical priority*:An interrupt's logical priority describes that interrupt's precedence over other interrupts.

The relationship between an interrupt's numeric priority and logical priority is dependent on `the processor architecture`.

A *full interrupt nesting model* is created by setting `configMAX_SYSCALL_INTERRUPT_PRIORITY `to a higher logical interrupt priority than `configKERNEL_INTERRUPT_PRIORITY`.

- Interrupts that use priorities `1` to `3`, inclusive, are *prevented* from executing while the kernel or the application is inside a *critical section*.
- Interrupts that use priority `4`, or `above`, are `not affected` by critical sections, so nothing the scheduler does will prevent these interrupts from executing immediately—within the limitations of the hardware itself. ISRs executing at these priorities `cannot use` any `FreeRTOS API` functions.
- Typically, functionality that requires *very strict timing accuracy* (motor control, for example) would use a priority above `configMAX_SYSCALL_INTERRUPT_PRIORITY` to ensure the scheduler does not introduce `jitter` into the interrupt response time.
#### A Note to ARM Cortex-M[^1] and ARM GIC Users
[^1]: This section only partially applies to Cortex-M0 and Cortex-M0+ cores.
- The ARM `Cortex` cores, and ARM Generic Interrupt Controllers (GICs), use numerically *low priority* numbers to represent logically *high priority* interrupts. This can seem counter-intuitive, and is easy to forget.
- The `Cortex-M` interrupt controller allows a maximum of eight bits to be used to specify each interrupt priority, making `255` the lowest possible priority. `Zero` is the `highest` priority. However, Cortex-M microcontroller

Normally only implement a subset of the `eight possible bits`. The number of bits actually implemented is dependent on the microcontroller family.

When only a `subset` of the eight possible bits has been implemented, it is only *the most significant bits* of the *byte* that can be used—leaving the least significant bits unimplemented. `Unimplemented bits` can take any value, but it is normal to set them to `1`.

`Cortex-M` interrupts will *default* to a priority of `zero`—the highest possible priority. The implementation of the Cortex-M hardware does not permit `configMAX_SYSCALL_INTERRUPT_PRIORITY` to be set to *0*, so the priority of an interrupt that uses the `FreeRTOS API` must `never` be left at its `default value`.
## Resource Management
### Introduction and Scope
- *Non-atomic Access to Variables*:Updating `multiple members of a structure`, or updating a `variable `that is `larger than the natural word size` of the architecture (for example, updating a 32-bit variable on a 16-bit machine)
- *Function Reentrancy*:`Reentrant functions` are said to be `'thread safe'` because they can be accessed from more than one thread of execution without the risk of `data` or `logical operations` becoming corrupted.If a function does *not access* any data `other than` data *stored on the stack* or *held in a register*, then the function is reentrant, and thread safe.
#### Mutual Exclusion
To ensure *data consistency* is maintained at all times, access to a resource that is shared `between tasks`, or is shared `between tasks and interrupts`, must be managed using a `'mutual exclusion'` technique.

`Shared resource` that is `not re-entrant` and `not thread-safe`.

#### Scope
- [ ] When and why resource management and control is necessary.
- [ ] What a critical section is.
- [ ] What mutual exclusion means.
- [ ] What it means to suspend the scheduler.
- [ ] How to use a mutex.
- [ ] How to create and use a gatekeeper task.
- [ ] What priority inversion is, and how priority inheritance can reduce (but not remove) its impact.
### Critical Sections and Suspending the Scheduler
#### Basic Critical Sections
Basic critical sections are `regions of code` that are surrounded by calls to the macros `taskENTER_CRITICAL()` and `taskEXIT_CRITICAL()`, respectively.

- Basic critical sections must be `kept very short`, otherwise they will adversely affect *interrupt response times*.
- Every call to `taskENTER_CRITICAL()` must be closely `paired` with a call to `taskEXIT_CRITICAL()`.
- It is safe for `critical sections` to become **nested**, because the kernel keeps a *count* of the `nesting depth`.
#### Suspending (or Locking) the Scheduler
`Critical sections` can also be created by suspending the scheduler. Suspending the scheduler is sometimes also known as `'locking'` the scheduler.
- A critical section implemented by suspending the scheduler `only protects` a region of code from access by `other tasks`, because *interrupts* remain enabled.
- A critical section that is `too long` to be implemented by `simply disabling interrupts` can, instead, be implemented by suspending the scheduler.
#### vTaskSuspendAll()
Suspending the scheduler `prevents `a *context switch* from occurring, but leaves `interrupts enabled`.
- `FreeRTOS API` functions must not be called while the `scheduler` is suspended.
- It is safe for calls to `vTaskSuspendAll()` and `xTaskResumeAll()` to become **nested**, because the kernel keeps a *count* of the `nesting depth`.
### Mutexes (and Binary Semaphores)
A Mutex is a `special type` of binary semaphore that is used to `control access` to a resource that is shared between `two or more tasks`.
- `configUSE_MUTEXES` must be set to `1` in `FreeRTOSConfig.h` for mutexes to be available.
- When used in a *mutual exclusion scenario*, the mutex can be thought of as a `token` that is associated with the resource being shared.

The primary difference is what happens to the semaphore after it has been obtained:
- A `semaphore` that is used for `mutual exclusion` must always be `returned`.
- A `semaphore` that is used for `synchronization` is normally *discarded* and *not returned*.
#### xSemaphoreCreateMutex()
A `mutex` is a type of `semaphore`. Handles to all the various types of FreeRTOS semaphore are stored in a variable of type `SemaphoreHandle_t`.
#### Priority Inversion
> Priority inheritance is a scheme that minimizes the negative effects of `priority inversion`.

- FreeRTOS `mutexes` and `binary semaphores` are very similar—the difference being that mutexes include a basic `'priority inheritance'` mechanism, whereas binary semaphores do not.
- *Priority inheritance* works by `temporarily raising` the priority of the mutex holder to the priority of `the highest priority task` that is *attempting to obtain* the same mutex.
- The priority of the mutex holder is *reset* automatically to its original value when it `gives the mutex` back.
- Mutexes must *not* be used from `interrupt service routines`.

Specific behaviors of the priority inheritance mechanism to keep in mind:
- A task can have its inherited priority `raised` further if it takes a mutex without first releasing mutexes it already holds.
- A task remains at its `highest` inherited priority until it has released `all the mutexes` it holds. This is regardless of the order the mutexes are released.
- A task will `remain` at the highest inherited priority if multiple mutexes are held regardless of tasks waiting on any of the held mutexes completing their wait (timing out).
#### Deadlock (or Deadly Embrace)
> `'Deadlock'` is `another potential pitfall` of using mutexes for mutual exclusion.

- Deadlock occurs when `two tasks` cannot proceed because they are both waiting for a *resource* that is held by the other.
- Instead, use a **time out** that is a little longer than the maximum time it is expected to have to wait for the mutex
#### Recursive Mutexes
It is also possible for a task to `deadlock` with itself. This will happen if a task attempts to take `the same mutex` *more than once*, without first `returning` the mutex.

Consider the following scenario:
1. A task successfully obtains a mutex.
2. While holding the mutex, the task calls a library function.
3. The implementation of the library function attempts to take the same mutex, and enters the Blocked state to wait for the mutex to become available.

- A `recursive mutex` can be 'taken' more than once by the same task, and will be returned only after one call to 'give' the recursive mutex has been executed for every preceding call to 'take' the recursive mutex.
#### Mutexes and Task Scheduling
*`two tasks` of `different` priority use the `same mutex`*
- The high priority task will **preempt** the low priority task as soon as the low priority task `returns the mutex`.

*`two tasks` of `same` priority use the `same mutex`*
- Task 1 will **not preempt** Task 2 when Task 2 'gives' the mutex. Instead, Task 2 will remain in the `Running` state, and Task 1 will simply move from the `Blocked` state to the *Ready* state.
- If a task uses a mutex in a `tight loop`, and a `context switch` occurred each time the task 'gave' the mutex, then the task would only ever remain in the `Running state` for a short time. If `two or more tasks` used the same mutex in a tight loop, then processing time would be *wasted by rapidly switching between the tasks.*

> The wasted time can be avoided by adding a call to `taskYIELD()` after the call to `xSemaphoreGive()`.
```C
void vFunction( void *pvParameter ) {
    extern SemaphoreHandle_t xMutex;
    char cTextBuffer[ 128 ];
    TickType_t xTimeAtWhichMutexWasTaken;

    for( ;; ) {
        /* Generate the text string – this is a fast operation. */
        vGenerateTextInALocalBuffer( cTextBuffer );

        /* Obtain the mutex that is protecting access to the display. */
        xSemaphoreTake( xMutex, portMAX_DELAY );

        /* Record the time at which the mutex was taken. */
        xTimeAtWhichMutexWasTaken = xTaskGetTickCount();

        /* Write the generated text to the display – this is a slow operation. */
        vCopyTextToFrameBuffer( cTextBuffer );

        /* The text has been written to the display, so return the mutex. */
        xSemaphoreGive( xMutex );

        /* If taskYIELD() was called on each iteration then this task would
           only ever remain in the Running state for a short period of time,
           and processing time would be wasted by rapidly switching between tasks.
           Therefore, only call taskYIELD() if the tick count changed while the
           mutex was held. */
        if( xTaskGetTickCount() != xTimeAtWhichMutexWasTaken ) {
            taskYIELD();
        }
    }
}
```
### Gatekeeper Tasks
`Gatekeeper tasks` provide a `clean method` of *implementing mutual* exclusion without the risk of `priority inversion` or `deadlock`.

- A gatekeeper task is a task that has **sole ownership of a resource**. Only the gatekeeper task is allowed to access the resource `directly`—any other task needing to access the resource can do so only `indirectly` by using `the services of the gatekeeper`.

A `tick hook` (or tick callback) is a function that is called by the `kernel `during `each tick interrupt`. To use a tick hook function:
- Set `configUSE_TICK_HOOK` to 1 in FreeRTOSConfig.h.
- Provide the `implementation` of the hook function, using `the exact function name` and `prototype` shown
```C
void vApplicationTickHook( void );
```

`Tick hook functions` execute within the *context* of the **tick interrupt**.
- The `scheduler` will always execute `immediately` after the `tick hook function`, so interrupt safe FreeRTOS API functions called from the tick hook do not need to use their `pxHigherPriorityTaskWoken` parameter, and that parameter can be set to `NULL`.

In some situations, it would be appropriate to assign the `gatekeeper` a higher priority, so messages get processed immediately—but doing so would be at `the cost` of the gatekeeper `delaying` lower priority tasks until it has completed `accessing the protected resource`.
## Event Groups
### Introduction and Scope
`Semaphores` and `queues`, both of which have the following properties:
- They allow a task to wait in the `Blocked` state for *a single event* to occur.
- They `unblock` a single task when the event occurs. The task that is unblocked is the `highest priority` task that was waiting for the event.

`Event groups` are another feature of FreeRTOS that `allow events` to be `communicated` to `tasks`. Unlike `queues` and `semaphores`:
- `Event groups` allow a task to wait in the `Blocked` state for *a combination of one of more events* to occur.
- `Event groups` unblock *all the tasks* that were waiting for `the same event, or combination of events`, when the event occurs.

These unique properties of event groups make them useful for `synchronizing` multiple tasks, `broadcasting events` to more than one task, allowing a task to wait in the Blocked state for `any one of a set of events` to occur, and allowing a task to wait in the Blocked state for `multiple actions to complete`.

`Event groups` also provide the opportunity to reduce the `RAM` used by an application as, often, it is possible to *replace* `many binary semaphores` with a single event group.
#### Scope
- [ ] Practical uses for event groups.
- [ ] The advantages and disadvantages of event groups relative to other FreeRTOS features.
- [ ] How to set `bits` in an event group.
- [ ] How to wait in the Blocked state for bits to become set in an event group.
- [ ] How to use an event group to `synchronize` a set of tasks.
### Characteristics of an Event Group
#### Event Groups, Event Flags and Event Bits
> An `event 'flag'` is a `Boolean` (1 or 0) value used to indicate if an event has occurred or not. An `event 'group'` is a set of event flags.

- The state of each `event flag` in an event group is represented by `a single bit` in a variable of type `EventBits_t`.
- If a bit is set to `1` in the `EventBits_t` variable, then the event represented by that bit has *occurred*.
- It is up to the `application writer` to assign a meaning to `individual bits` within an event group.
#### More About the EventBits_t Data Type
- `configTICK_TYPE_WIDTH_IN_BITS` configures the type used to hold the `RTOS tick count`, so would seem unrelated to the event groups feature. Its effect on the `EventBits_t` type is a consequence of `FreeRTOS's internal implementation`.
- If `configTICK_TYPE_WIDTH_IN_BITS` is `TICK_TYPE_WIDTH_16_BITS`, then each event group contains **8** usable event bits.
- If `configTICK_TYPE_WIDTH_IN_BITS` is `TICK_TYPE_WIDTH_32_BITS`, then each event group contains **24** usable event bits.
- If `configTICK_TYPE_WIDTH_IN_BITS` is `TICK_TYPE_WIDTH_64_BITS`, then each event group contains **56** usable event bits.
#### Access by Multiple Tasks
> `Event groups` are `objects` in their own right that can be accessed by any `task` or `ISR` that knows of their existence.

#### A Practical Example of Using an Event Group
The state of a *FreeRTOS+TCP* `socket` is held in a structure called `FreeRTOS_Socket_t`. The structure contains an `event group` that has an event bit defined for each event the socket must process.

The event group also contains an `'abort'` bit, allowing a TCP connection to be `aborted`, no matter which event the socket is waiting for at the time.
### Event Management Using Event Groups
#### xEventGroupSetBits()
The `xEventGroupSetBits()` API function `sets` `one or more bits` in an event group.
- It is typically used to *notify* a task that the events represented by the bit, or bits, being set `has occurred`.
```C
/**
 * @brief Sets bits within an event group.
 *
 * This function sets bits within an event group to the value 1. It can be used to
 * signal that an event or events have occurred. The event group can be created using
 * the xEventGroupCreate() function.
 *
 * @param xEventGroup The handle to the event group in which bits are being set.
 *                    This handle is returned from a previous call to
 *                    xEventGroupCreate().
 * @param uxBitsToSet A bitwise value that specifies the event bit or bits to set.
 *                    The event group's value is updated by bitwise `ORing` the
 *                    event group's existing value with the value passed in
 *                    uxBitsToSet.
 *
 * @return EventBits_t The value of the event group at the time the call to
 *                     xEventGroupSetBits() returned. Note that the returned value
 *                     may not have the bits specified by uxBitsToSet set, because
 *                     the bits may have been cleared again by a different task.
 *
 * @note As an example, setting uxBitsToSet to 0x04 (binary 0100) will result in
 *       event bit 3 in the event group becoming set (if it was not already set),
 *       while leaving all the other event bits in the event group unchanged.
 */
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup, const EventBits_t uxBitsToSet );
```
#### xEventGroupSetBitsFromISR()
Giving a `semaphore` is a *deterministic operation* because it is known in advance that giving a semaphore can result in `at most one task` leaving the Blocked state. When bits are set in an `event group` it is `not known` in advance how many tasks will leave the Blocked state, so setting bits in an event group is *not a deterministic operation*.

The `FreeRTOS` design and implementation standard does not permit *non-deterministic operations* to be performed inside an `interrupt service routine`, or when `interrupts are disabled`.

> `xEventGroupSetBitsFromISR()` does not set event bits `directly` inside the interrupt service routine, but instead defers the action to the `RTOS daemon task`.
```C
/**
 * @brief Sets bits within an event group from an interrupt service routine (ISR).
 *
 * This function sets bits within an event group to the value 1, from within an ISR.
 * It does not set the event bits directly inside the ISR, but instead defers the
 * action to the RTOS daemon task by sending a command on the timer command queue.
 * If the daemon task was in the Blocked state to wait for data to become available
 * on the timer command queue, then writing to the timer command queue will cause
 * the daemon task to leave the Blocked state.
 *
 * @param xEventGroup The handle to the event group in which bits are being set.
 *                    This handle is returned from a previous call to
 *                    xEventGroupCreate().
 * @param uxBitsToSet A bitwise value that specifies the event bit or bits to set.
 *                    The event group's value is updated by bitwise ORing the
 *                    event group's existing value with the value passed in
 *                    uxBitsToSet.
 * @param pxHigherPriorityTaskWoken A pointer to a variable that will be set to
 *                                  pdTRUE if sending the command to the timer
 *                                  command queue caused the RTOS daemon task to
 *                                  unblock, and the priority of the daemon task is
 *                                  higher than the priority of the currently
 *                                  executing task (the task that was interrupted).
 *                                  If xEventGroupSetBitsFromISR() sets this
 *                                  value to pdTRUE, then a context switch should be
 *                                  performed before the interrupt is exited.
 *
 * @return BaseType_t
 * - pdPASS: Data was successfully sent to the timer command queue.
 * - pdFALSE: The 'set bits' command could not be written to the timer command
 *            queue because the queue was already full.
 *
 * @note As an example, setting uxBitsToSet to 0x05 (binary 0101) will result in
 *       event bit 2 and event bit 0 in the event group becoming set (if they were
 *       not already set), while leaving all the other event bits in the event group
 *       unchanged.
 */
BaseType_t xEventGroupSetBitsFromISR(
    EventGroupHandle_t xEventGroup,
    const EventBits_t uxBitsToSet,
    BaseType_t *pxHigherPriorityTaskWoken
);
```
#### xEventGroupWaitBits()
The `xClearOnExit` parameter is provided to avoid these `potential race conditions`. If `xClearOnExit` is set to `pdTRUE`, then the testing and clearing of event bits appears to the calling task to be an *atomic operation* (uninterruptible by other tasks or interrupts).
```C
/**
 * @brief Waits for one or more bits to be set within an event group.
 *
 * This function allows a task to wait for one or more bits to be set within an event
 * group. The task will either block until the bits are set or until a specified block
 * time expires, depending on the value of xTicksToWait.
 *
 * @param xEventGroup The handle to the event group that contains the event bits
 *                    being read. This handle is returned from a previous call to
 *                    xEventGroupCreate().
 * @param uxBitsToWaitFor A bit mask that specifies the event bit or event bits to
 *                        test in the event group.
 * @param xClearOnExit If the calling task's unblock condition has been met and
 *                    xClearOnExit is set to pdTRUE, then the event bits specified
 *                    by uxBitsToWaitFor will be cleared back to 0 in the event
 *                    group before the calling task exits the xEventGroupWaitBits()
 *                    API function.
 * @param xWaitForAllBits If set to pdTRUE, the task will only be unblocked when
 *                        all bits specified by uxBitsToWaitFor are set. If set
 *                        to pdFALSE, the task will be unblocked when any of the
 *                        bits specified by uxBitsToWaitFor are set.
 * @param xTicksToWait The maximum amount of time the task should remain in the
 *                     Blocked state to wait for its unblock condition to be met.
 *                     The block time is specified in tick periods.
 *
 * @return EventBits_t The value of the event group at the time the calling task's
 *                     unblock condition was met (before any bits were automatically
 *                     cleared if xClearOnExit was pdTRUE). If the function returned
 *                     because the block time expired, the returned value is the
 *                     value of the event group at the time the block time expired.
 *                     In this case, the returned value will not meet the unblock
 *                     condition.
 *
 * @note If xEventGroupWaitBits() returned because the calling task's unblock
 *       condition was met, then the returned value will also meet the unblock
 *       condition. If xEventGroupWaitBits() returned because the block time
 *       specified by the xTicksToWait parameter expired, then the returned value
 *       is the value of the event group at the time the block time expired.
 */
EventBits_t xEventGroupWaitBits(
    EventGroupHandle_t xEventGroup,
    const EventBits_t uxBitsToWaitFor,
    const BaseType_t xClearOnExit,
    const BaseType_t xWaitForAllBits,
    TickType_t xTicksToWait
);
```
### Task Synchronization Using an Event Group
An event group can be used to create a `synchronization point`:
- `Each task` that must participate in the `synchronization` is assigned a `unique event bit` within the event group.
- `Each task` `sets` its own event bit when it reaches the synchronization point.
- Having set its own event bit, each task `blocks` on the event group to wait for the event bits that represent all the other synchronizing tasks to also become set.

To successfully use an `event group` to create a `synchronization point`, the setting of an event bit, and the `subsequent testing` of event bits, must be performed as a *single uninterruptible operation*.The `xEventGroupSync()` API function is provided for that purpose.
#### xEventGroupSync()
The function allows a task to set one or more event bits in an event group, then wait for *a combination of event bits* to become set in the same event group, as `a single uninterruptable operation`.
```C
/**
 * @brief Synchronizes with event flags or waits for event flag bits to be set.
 *
 * This function sets bits in an event group, then waits for a set of bits to be set.
 * The task will block until the bits are set, or until a specified timeout period expires.
 * The bits specified by uxBitsToWaitFor will be cleared back to zero before 
 * xEventGroupSync() returns, if the unblock condition has been met.
 *
 * @param xEventGroup The handle to the event group in which event bits are to be
 *                    set and then tested. This handle is returned from a previous
 *                    call to xEventGroupCreate().
 * @param uxBitsToSet A bit mask that specifies the event bit or event bits to set
 *                    to 1 in the event group. The event group's value is updated
 *                    by bitwise ORing the event group's existing value with the
 *                    value passed in uxBitsToSet.
 * @param uxBitsToWaitFor A bit mask that specifies the event bit or event bits
 *                        to test in the event group.
 * @param xTicksToWait The maximum amount of time the task should remain in the
 *                     Blocked state to wait for its unblock condition to be met.
 *                     The block time is specified in tick periods.
 *
 * @return EventBits_t If xEventGroupSync() returned because the calling task's
 *                     unblock condition was met, then the returned value is the
 *                     value of the event group at the time the calling task's
 *                     unblock condition was met (before any bits were automatically
 *                     cleared back to zero). If xEventGroupSync() returned because
 *                     the block time specified by the xTicksToWait parameter expired,
 *                     then the returned value is the value of the event group at
 *                     the time the block time expired. In this case, the returned
 *                     value will not meet the calling task's unblock condition.
 *
 * @note Setting xTicksToWait to portMAX_DELAY will cause the task to wait
 *       indefinitely (without timing out), provided INCLUDE_vTaskSuspend is set
 *       to 1 in FreeRTOSConfig.h.
 */
EventBits_t xEventGroupSync(
    EventGroupHandle_t xEventGroup,
    const EventBits_t uxBitsToSet,
    const EventBits_t uxBitsToWaitFor,
    TickType_t xTicksToWait
);
```
## Task Notifications
### Introduction
`Task notifications` are an efficient mechanism allowing one task to *directly notify* another task.
#### Communicating Through Intermediary Objects
The methods described so far have required the creation of a *communication object*. Examples of communication objects include `queues`, `event groups`, and `various different types of semaphore`.
#### Task Notifications—Direct to Task Communication
`Task Notifications` allow tasks to interact with `other tasks`, and to `synchronize with ISR`s, `without` the need for a *separate communication object*.

When `configUSE_TASK_NOTIFICATIONS` is set to `1`, each task has at least one `Notification State`, which can be either *Pending* or *Not-Pending*, and a *Notification Value*, which is a `32-bit unsigned integer`.
- When a task receives a notification, its notification state is set to `pending`.
- When a task reads its `notification value`, its notification state is set to `not-pending`.
- A task can wait in the `Blocked` state, with an optional `time out`, for its notification state to become `pending`.
#### Scope
- [ ] A task's notification state and notification value.
- [ ] How and when a task notification can be used in place of a `communication object`, such as a semaphore.
- [ ] The `advantages` of using a task notification in place of a communication object.
### Task Notifications; Benefits and Limitations
#### Performance Benefits of Task Notifications
Using a `task notification` to send an *event or data* to a task is `significantly faster` than using a `queue`, `semaphore` or `event group` to perform an equivalent operation.
#### RAM Footprint Benefits of Task Notifications
`Each communication object `(queue, semaphore or event group) must be `created` before it can be used, whereas enabling `task notification` functionality has a `fixed overhead`.
- The default value for `configTASK_NOTIFICATION_ARRAY_ENTRIES` is `1` making the `default` size for task notifications is **5 bytes** `per task`.
#### Limitations of Task Notifications
`Task notifications` are *faster* and *use less RAM* than `communication objects`, but `task notifications` cannot be used in `all scenarios`. This section documents the scenarios in which a task notification cannot be used:
- *Sending an `event` or `data` to an `ISR`*: Task notifications can be used to send events and data `from an ISR to a task`, but they `cannot` be used to send events or data `from a task to an ISR`.
- *Enabling `more than one` receiving task*
- *Buffering `multiple data items`*: Task notifications send data to a task by **updating** the receiving task's `notification value`.
- *Broadcasting to more than one task*
- *Waiting in the `blocked` state for a send to complete*: If a task attempts to send a task notification to a task that already has a `notification pending`, then it is `not` possible for the sending task to `wait` in the `Blocked` state for the receiving task to reset its notification state.
### Using Task Notifications
#### Task Notification API Options
`Task notifications` are a very powerful feature that can often be used in place of a `binary semaphore`, a `counting semaphore,` an `event group`, and sometimes even a `queue`.
- Using the `xTaskNotify()` API function to *send* a task notification, and the `xTaskNotifyWait()` API function to *receive* a task notification.
- The `xTaskNotifyGive()` API function is provided as a *simpler but less flexible* alternative to `xTaskNotify()`, and the `ulTaskNotifyTake()` API function is provided as a simpler but less flexible alternative to `xTaskNotifyWait()`.
- The configuration parameter `configTASK_NOTIFICATION_ARRAY_ENTRIES` is set to `1` by default. If it is set to a value *greater than 1*, an `array` of notifications are created inside each task. This allows notifications to be managed by `index`.

The `task notification API's` are implemented as *macro* that make calls to the `underlying Generic versions` of each API function type.
#### xTaskNotifyGive()
```C
/**
 * @brief Sends a task notification to a task, which can be used as an alternative to
 * a binary or counting semaphore.
 *
 * This function sends a notification to a task, without modifying the task's
 * notification value. It is a simpler and faster method of inter-task communication
 * compared to using binary or counting semaphores. The notification is done by
 * unblocking the task directly if it is in the Blocked state, or by setting the
 * task's notification state if it is not blocked.
 *
 * @param xTaskToNotify The handle of the task to which the notification is being sent.
 *                      This handle is obtained as the return value of the xTaskCreate()
 *                      API function when the task is created.
 *
 * @return BaseType_t Always returns pdPASS, indicating that the notification was sent
 *                   successfully.
 *
 * @note This function is a macro that calls xTaskNotify() with parameters set so that
 *       pdPASS is the only possible return value.
 */
BaseType_t xTaskNotifyGive(TaskHandle_t xTaskToNotify);
```
#### vTaskNotifyGiveFromISR()
```C
/**
 * @brief Sends a task notification to a task from an interrupt service routine (ISR).
 *
 * This function is an ISR-safe version of xTaskNotifyGive(). It is used to send a
 * notification to a task, which can be used as an alternative to a binary or counting
 * semaphore. If the notified task leaves the Blocked state, a context switch may be
 * required depending on the priority of the unblocked task.
 *
 * @param xTaskToNotify The handle of the task to which the notification is being sent.
 *                      This handle is obtained as the return value of the xTaskCreate()
 *                      API function when the task is created.
 * @param pxHigherPriorityTaskWoken A pointer to a variable that will be set to pdTRUE
 *                                  if sending the notification caused a task to
 *                                  leave the Blocked state and the unblocked task
 *                                  has a higher priority than the currently executing
 *                                  task (the task that was interrupted). If this is the
 *                                  case, a context switch should be performed before
 *                                  the interrupt is exited to ensure the highest priority
 *                                  Ready state task runs next.
 *
 * @note As with all interrupt safe API functions, the pxHigherPriorityTaskWoken
 *       parameter must be initialized to pdFALSE before it is used.
 */
void vTaskNotifyGiveFromISR(TaskHandle_t xTaskToNotify, BaseType_t *pxHigherPriorityTaskWoken);
```
#### ulTaskNotifyTake()
`ulTaskNotifyTake()` allows a task to wait in the `Blocked` state for its `notification value` to be *greater than zero*, and either decrements (subtracts one from) or `clears` the task's notification value `before` it returns.
```C
/**
 * @brief Retrieves the task's notification value and optionally clears it.
 *
 * This function provides a mechanism to receive a task notification, which can be
 * used as a lighter weight and faster alternative to a binary or counting semaphore.
 * The function can either clear the notification value to zero or decrement it,
 * depending on the parameter provided.
 *
 * @param xClearCountOnExit If set to pdTRUE, the calling task's notification value
 *                          is cleared to zero before the function returns.
 *                          If set to pdFALSE, and the task's notification value is
 *                          greater than zero, the notification value is decremented
 *                          before the function returns.
 * @param xTicksToWait The maximum amount of time the calling task should remain
 *                     in the Blocked state to wait for its notification value to
 *                     be greater than zero. This is specified in tick periods, and
 *                     the macro pdMS_TO_TICKS() can be used to convert a time
 *                     specified in milliseconds to ticks.
 *
 * @return uint32_t The calling task's notification value before it was either cleared
 *                  to zero or decremented, as specified by xClearCountOnExit.
 *                  If the block time was specified and the return value is not zero,
 *                  it is possible that the calling task was placed into the Blocked
 *                  state and its notification value was updated before the block time
 *                  expired. If the block time was specified and the return value is zero,
 *                  the calling task was placed into the Blocked state but the block time
 *                  expired before the notification value became greater than zero.
 *
 * @note Setting xTicksToWait to portMAX_DELAY will cause the task to wait indefinitely
 *       (without timing out), provided INCLUDE_vTaskSuspend is set to 1 in FreeRTOSConfig.h.
 */
uint32_t ulTaskNotifyTake(BaseType_t xClearCountOnExit, TickType_t xTicksToWait);
```
#### xTaskNotify()
> `xTaskNotify()` is a `more capable` version of `xTaskNotifyGive()` that can be used to `update` the receiving task's notification value in any of the following ways:
- *Increment* (add one to) the receiving task's notification value, in which case `xTaskNotify()` is equivalent to `xTaskNotifyGive()`.
- Set *one or more bits* in the receiving task's `notification value`. This allows a task's notification value to be used as a `lighter weight and faster alternative` to an **event group**.
- Write a completely *new number* into the receiving task's notification value, but only if the receiving task `has read` its notification value since it was last updated. This allows a task's notification value to provide `similar functionality` to that provided by a **queue** that has a length of `one`.
- Write a completely *new number* into the receiving task's notification value, even if the receiving task `has not read` its notification value since it was last updated. This allows a task's notification value to provide similar functionality to that provided by the `xQueueOverwrite()` API function. The resultant behavior is sometimes referred to as a **'mailbox'**.

`xTaskNotifyFromISR()` is a version of `xTaskNotify()` that can be used in an `interrupt service routine`, and therefore has an additional `pxHigherPriorityTaskWoken` parameter.
```C
/**
 * @brief Sends a notification to a task.
 *
 * This function sends a notification to a task, allowing various actions to be
 * specified which determine how the task's notification value is updated.
 *
 * @param xTaskToNotify The handle of the task to which the notification is being sent.
 *                      This handle is obtained as the return value of the xTaskCreate()
 *                      API function when the task is created.
 * @param ulValue The value to use when updating the receiving task's notification
 *                value, depending on the value of eAction.
 * @param eAction An enumerated type that specifies how to update the receiving
 *                task's notification value.
 *
 * @return BaseType_t pdPASS if the notification was sent successfully, pdFAIL if
 *                  no action was taken because the task already had a notification
 *                  pending and eAction was eSetValueWithoutOverwrite.
 *
 * @note Valid eNotifyAction values and their effects on the receiving task's
 *       notification value are:
 *       - eNoAction: The task's notification state is set to pending without its
 *                   notification value being updated. The ulValue parameter is not used.
 *       - eSetBits: The task's notification value is bitwise OR'ed with ulValue.
 *       - eIncrement: The task's notification value is incremented. ulValue is not used.
 *       - eSetValueWithoutOverwrite: If a notification is pending, pdFAIL is returned.
 *                                     Otherwise, the task's notification value is set
 *                                     to ulValue.
 *       - eSetValueWithOverwrite: The task's notification value is set to ulValue,
 *                                  regardless of any pending notifications.
 */
BaseType_t xTaskNotify(TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction);
```
#### xTaskNotifyWait()
`xTaskNotifyWait()` is a more capable version of `ulTaskNotifyTake()`. It allows a task to wait, with an optional *timeout*, for the calling task's notification state to become pending, should it not already be pending. `xTaskNotifyWait()` provides options for bits to be cleared in the calling task's notification value `both` on `entry` to the function, and on `exit` from the function.
```C
/**
 * @brief Waits for a task notification.
 *
 * This function allows a task to wait for a notification to be sent to it, optionally
 * clearing specific bits on entry and exit, and saving the notification value.
 *
 * @param ulBitsToClearOnEntry If the task did not have a notification pending before
 *                             calling xTaskNotifyWait(), then any bits set in
 *                             ulBitsToClearOnEntry will be cleared in the task's
 *                             notification value on entry to the function.
 * @param ulBitsToClearOnExit If the task exits xTaskNotifyWait() because it received
 *                            a notification, or because it already had a notification
 *                            pending when xTaskNotifyWait() was called, then any bits
 *                            set in ulBitsToClearOnExit will be cleared in the task's
 *                            notification value before the task exits the function.
 * @param pulNotificationValue A pointer to a variable that will be used to store the
 *                              task's notification value before any bits are cleared
 *                              due to the ulBitsToClearOnExit setting. This parameter
 *                              is optional and can be set to NULL if not required.
 * @param xTicksToWait The maximum amount of time the task should remain in the Blocked
 *                     state to wait for its notification state to become pending. The
 *                     block time is specified in tick periods.
 *
 * @return BaseType_t
 * - pdTRUE if xTaskNotifyWait() returned because a notification was received, or
 *           because the task already had a notification pending when it was called.
 * - pdFALSE if xTaskNotifyWait() returned without the task receiving a notification,
 *           and the block time expired.
 *
 * @note Setting xTicksToWait to portMAX_DELAY will cause the task to wait indefinitely
 *       (without timing out), provided INCLUDE_vTaskSuspend is set to 1 in FreeRTOSConfig.h.
 */
BaseType_t xTaskNotifyWait(uint32_t ulBitsToClearOnEntry,
                           uint32_t ulBitsToClearOnExit,
                           uint32_t *pulNotificationValue,
                           TickType_t xTicksToWait);
```
### Task Notifications Used in Peripheral Device Drivers: ADC Example
`vTaskNotifyGiveFromISR()` is a simple function to use, but its capabilities are limited; it can only send a task notification as a `valueless event`, it cannot *send data*. This section demonstrates how to use `xTaskNotifyFromISR()` to `send data` with `a task notification event`.
### Task Notifications Used Directly Within an Application ★
## Low
#待定
## Developer Support
### Introduction
This chapter highlights a set of *features* that are included to `maximize productivity` by:
- Providing insight into how an application is behaving.
- Highlighting opportunities for optimization.
- Trapping errors at the point at which they occur.
### configASSERT()
In C, the macro `assert()` is used to verify an assertion (an assumption) made by the program. The assertion is written as a C expression, and if the expression evaluates to false (0), then the assertion has deemed to have failed.

The FreeRTOS source code does not call assert(), because assert() is not available with all the compilers with which FreeRTOS is compiled. Instead, the FreeRTOS source code contains lots of calls to a macro called `configASSERT()`, which can be defined by the application writer in FreeRTOSConfig.h, and behaves exactly like the standard C assert().

A failed assertion must be treated as a fatal *error*. Do not attempt to execute `past a line` that has failed an assertion.
### Tracealyzer for FreeRTOS
Tracealyzer for FreeRTOS captures valuable dynamic behavior information, then presents the captured information in interconnected graphical views. The tool is also capable of displaying multiple synchronized views.
### Debug Related Hook (Callback) Functions
#### Malloc failed hook
Defining a malloc failed hook ensures the application developer is notified immediately if an attempt to create a task, queue, semaphore or event group fails.
#### Stack overflow hook
Defining a stack overflow hook ensures the application developer is notified if the amount of stack used by a task exceeds the stack space allocated to the task.
### Viewing Run-time and Task State Information
#### Task Run-Time Statistics
`Task run-time statistics` provide information on the amount of processing time each task has received. A task's run time is the total time the task has been in the `Running` state since the application `booted`.
#### The Run-Time Statistics Clock
`Run-time statistics` need to measure fractions of a tick period. Therefore, the `RTOS tick count` is not used as the run-time statistics clock, and the clock is instead provided by the `application code`. It is recommended to make the frequency of the run-time statistics clock between `10` and `100` times faster than the frequency of the tick interrupt. The faster the run-time statistics clock, the more accurate the statistics will be, but also the sooner the time value will overflow.

Ideally, the time value will be generated by a *free-running 32-bit peripheral timer/counter*, the value of which can be read with no other processing overhead.
#### Configuring an Application to Collect Run-Time Statistics
> `Macros` used in the collection of run-time statistics
- `configGENERATE_RUN_TIME_STATS`
- `portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`
- `portGET_RUN_TIME_COUNTER_VALUE(), or portALT_GET_RUN_TIME_COUNTER_VALUE(Time)`
#### uxTaskGetSystemState()
`uxTaskGetSystemState()` provides a `snapshot` of status information for each task under the control of the `FreeRTOS scheduler`.

- The information is provided as an *array* of `TaskStatus_t` structures, with one index in the array for each task.
```c
/**
 * @brief The uxTaskGetSystemState() API function prototype.
 * 
 * This function is used to retrieve the state of each task in the system.
 * 
 * @param pxTaskStatusArray A pointer to an array of TaskStatus_t structures.
 *                          The array must contain at least one TaskStatus_t
 *                          structure for each task. The number of tasks can be
 *                          determined using the uxTaskGetNumberOfTasks() API
 *                          function. 
 * 
 * @param uxArraySize The size of the array pointed to by the pxTaskStatusArray
 *                    parameter. The size is specified as the number of indexes
 *                    in the array (the number of TaskStatus_t structures
 *                    contained in the array), not by the number of bytes in the
 *                    array.
 * 
 * @param pulTotalRunTime If configGENERATE_RUN_TIME_STATS is set to 1 in
 *                        FreeRTOSConfig.h, then *pulTotalRunTime is set by
 *                        uxTaskGetSystemState() to the total run time (as
 *                        defined by the run-time statistics clock provided by
 *                        the application) since the target booted.
 *                        pulTotalRunTime is optional and can be set to NULL if
 *                        the total run time is not required.
 * 
 * @return The number of TaskStatus_t structures that were populated by
 *         uxTaskGetSystemState(). The returned value should equal the number
 *         returned by the uxTaskGetNumberOfTasks() API function, but will be
 *         zero if the value passed in the uxArraySize parameter was too small.
 * 
 * @note configRUN_TIME_COUNTER_TYPE defaults to uint32_t for backward
 *       compatibility, but can be overridden in FreeRTOSConfig.h if uint32_t is
 *       too restrictive.
 */
UBaseType_t uxTaskGetSystemState(
    TaskStatus_t * const pxTaskStatusArray,
    const UBaseType_t uxArraySize,
    configRUN_TIME_COUNTER_TYPE * const pulTotalRunTime
);
```

- `TaskStatus_t` structure members
```c
/**
 * @brief A structure that holds the status of a task.
 * 
 * This structure is used to return status information about each task.
 * 
 */
typedef struct xTASK_STATUS {
    /** The handle of the task to which the information in the structure relates. */
    TaskHandle_t xHandle;
    
    /** The human-readable text name of the task. */
    const char *pcTaskName;
    
    /** Each task has a unique xTaskNumber value. */
    UBaseType_t xTaskNumber;
    
    /**
     * @brief An enumerated type that holds the state of the task.
     * 
     * eCurrentState can be one of the following values:
     * - eRunning: The task is currently running.
     * - eReady: The task is ready to run but not currently running.
     * - eBlocked: The task is blocked waiting for a queue, semaphore, mutex, or binary semaphore.
     * - eSuspended: The task is suspended and will not be scheduled.
     * - eDeleted: The task has been deleted and is awaiting cleanup by the idle task.
     * 
     * A task will only be reported as being in the eDeleted state for the short period
     * between the time the task was deleted by a call to vTaskDelete(), and the time
     * the Idle task frees the memory that was allocated to the deleted task's internal
     * data structures and stack. After that time, the task will no longer exist in any way,
     * and it is invalid to attempt to use its handle.
     */
    eTaskState eCurrentState;
    
    /** The priority at which the task was running at the time uxTaskGetSystemState() was called. */
    UBaseType_t uxCurrentPriority;
    
    /** The priority assigned to the task by the application writer. */
    UBaseType_t uxBasePriority;
    
    /** The total run time used by the task since the task was created. */
    configRUN_TIME_COUNTER_TYPE ulRunTimeCounter;
    
    /** Points to the base address of the stack region allotted to this task. */
    StackType_t *pxStackBase;
    
    /**
     * Points to the current top address of the stack region allotted to this task.
     * This field is only valid if either the stack grows upwards (i.e. portSTACK_GROWTH is greater than zero)
     * or configRECORD_STACK_HIGH_ADDRESS is set to 1 in FreeRTOSConfig.h.
     */
    StackType_t *pxTopOfStack;
    
    /**
     * Points to the end address of the stack region allotted to this task.
     * This field is only valid if either the stack grows upwards (i.e. portSTACK_GROWTH is greater than zero)
     * or configRECORD_STACK_HIGH_ADDRESS is set to 1 in FreeRTOSConfig.h.
     */
    StackType_t *pxEndOfStack;
    
    /** The task's stack high water mark. */
    uint16_t usStackHighWaterMark;
    
    #if ( ( configUSE_CORE_AFFINITY == 1 ) && ( configNUMBER_OF_CORES > 1 ) )
    /**
     * A bitwise value that indicates the cores on which the task can run.
     * Cores are numbered from 0 to configNUMBER_OF_CORES - 1.
     * For example, a task that can run on core 0 and core 1 will have its uxCoreAffinityMask set to 0x03.
     * The field uxCoreAffinityMask is only available if both configUSE_CORE_AFFINITY is set to 1 and
     * configNUMBER_OF_CORES is set to greater than 1 in FreeRTOSConfig.h.
     */
    UBaseType_t uxCoreAffinityMask;
    #endif
} TaskStatus_t;
```
#### vTaskListTasks()
`TaskListTasks()` provides similar task status information to that provided by `uxTaskGetSystemState()`, but it presents the information as a human readable `ASCII table`, rather than an array of `binary values`.

`vTaskListTasks() `is a very *processor intensive* function, and leaves the *scheduler suspended* for an `extended period`. Therefore, it is recommended to use the function for `debug purposes only`, and not in a production real-time system.

vTaskListTasks() is available if `configUSE_TRACE_FACILITY` is set to `1` and `configUSE_STATS_FORMATTING_FUNCTIONS` is set to `greater than 0` in FreeRTOSConfig.h.
```c
/**
 * @brief Writes a list of all the currently running tasks into a provided buffer.
 *
 * @param pcWriteBuffer A pointer to a character buffer into which the formatted
 *                       and human-readable table is written. This buffer is
 *                       assumed to be large enough to contain the generated
 *                       report. Approximately 40 bytes per task should be
 *                       sufficient.
 * @param uxBufferLength The length of the pcWriteBuffer.
 */
void vTaskListTasks(char *pcWriteBuffer, size_t uxBufferLength);
```
#### vTaskGetRunTimeStatistics()
`vTaskGetRunTimeStatistics()` formats collected run-time statistics into a human readable `ASCII table`.

`vTaskGetRunTimeStatistics()` is a very *processor intensive* function and leaves the `scheduler suspended` for an `extended period`.

`vTaskGetRunTimeStatistics()` is available when `configGENERATE_RUN_TIME_STATS` is set to `1`, `configUSE_STATS_FORMATTING_FUNCTIONS` is set `greater than 0`, and `configUSE_TRACE_FACILITY` is set to `1` in FreeRTOSConfig.h.
```c
/**
 * @brief Writes a table of task runtime statistics into a provided buffer.
 *
 * @param pcWriteBuffer A pointer to a character buffer into which the formatted
 *                       and human-readable table of task runtime statistics is written.
 *                       This buffer is assumed to be large enough to contain the generated
 *                       report. Approximately 40 bytes per task should be sufficient.
 * @param uxBufferLength The length of the pcWriteBuffer.
 *
 * In the output:
 * - Each row provides information on a single task.
 * - The first column is the task name.
 * - The second column is the amount of time the task has spent in the Running state
 *   as an absolute value. See the description of ulRunTimeCounter for more details.
 * - The third column is the amount of time the task has spent in the Running state
 *   as a percentage of the total time since the target was booted.
 * - The total of the displayed percentage times will normally be less than the expected 100%
 *   because statistics are collected and calculated using integer calculations that round down
 *   to the nearest integer value.
 */
void vTaskGetRunTimeStatistics(char *pcWriteBuffer, size_t uxBufferLength);
```
### Trace Hook Macros
`Trace macros` are macros that have been placed at *key points* within the FreeRTOS source code.
By default, the macros are `empty`, and so do not generate any code, and have no run time overhead.
By overriding the default empty implementations, an application writer can:
- Insert code into FreeRTOS `without modifying` the FreeRTOS source files.
- `Output detailed execution sequencing information` by any means available on the target hardware. Trace macros appear in enough places in the FreeRTOS source code to allow them to be used to create a full and detailed scheduler activity trace and profiling log.
#### Available Trace Hook Macros
*pxCurrentTCB* is a FreeRTOS private variable that holds the handle of the task in the `Running state`, and is available to any macro that is called from the `FreeRTOS/Source/tasks.c` source file.

- *traceTASK_INCREMENT_TICK(xTickCount)* Called during the `tick interrupt`, before the tick count is incremented. The `xTickCount` parameter passes the new tick count value into the macro.
- *traceTASK_SWITCHED_OUT()* Called before a new task is selected to run. At this point, `pxCurrentTCB` contains the handle of the task about to leave the Running state.
- *traceTASK_SWITCHED_IN()* Called after a task is selected to run. At this point, `pxCurrentTCB` contains the handle of the task about to enter the Running state.
- *traceBLOCKING_ON_QUEUE_RECEIVE(pxQueue)* Called immediately before the `currently` executing task enters the `Blocked state` following an attempt to read from an `empty queue`, or an attempt to `'take'` an empty semaphore or mutex. The pxQueue parameter passes the handle of the target queue or semaphore into the macro.
- *traceBLOCKING_ON_QUEUE_SEND(pxQueue)* Called immediately before the currently executing task enters the Blocked state following an attempt to write to a queue that is `full.` The `pxQueue` parameter passes the handle of the target queue into the macro.
- *traceQUEUE_SEND(pxQueue)* Called from within `xQueueSend()`, `xQueueSendToFront()`,` xQueueSendToBack()`, or any of the semaphore 'give' functions, when the queue send or semaphore `'give'` is `successful`. The pxQueue parameter passes the handle of the target queue or semaphore into the macro
- *traceQUEUE_SEND_FAILED(pxQueue)* Called from within xQueueSend(), xQueueSendToFront(), xQueueSendToBack(), or any of the semaphore `'give'` functions, when the queue send or semaphore 'give' operation `fails.` A queue send or semaphore 'give' will fail if the queue is full and remains full for the duration of any block time specified. The pxQueue parameter passes the handle of the target queue or semaphore into the macro.
- *traceQUEUE_RECEIVE(pxQueue)* Called from within xQueueReceive() or any of the semaphore 'take' functions when the queue receive or semaphore 'take' is `successful`. The pxQueue parameter passes the handle of the target queue or semaphore into the macro.
- *traceQUEUE_RECEIVE_FAILED(pxQueue)* Called from within xQueueReceive() or any of the semaphore 'take' functions when the queue or semaphore receive operation fails. A queue receive or semaphore 'take' operation will fail if the queue or semaphore is empty and remains empty for the duration of any `block time` specified. The pxQueue parameter passes the handle of the target queue or semaphore into the macro.
- *traceQUEUE_SEND_FROM_ISR(pxQueue)* Called from within xQueueSendFromISR() when the send operation is successful. The pxQueue parameter passes the handle of the target queue into the macro.
- *traceQUEUE_SEND_FROM_ISR_FAILED(pxQueue)* Called from within xQueueSendFromISR() when the send operation fails. A send operation will fail if the queue is already full. The pxQueue parameter passes the handle of the target queue into the macro.
- *traceQUEUE_RECEIVE_FROM_ISR(pxQueue)* Called from within xQueueReceiveFromISR() when the receive operation is successful. The pxQueue parameter passes the handle of the target queue into the macro.
- *traceQUEUE_RECEIVE_FROM_ISR_FAILED(pxQueue)* Called from within xQueueReceiveFromISR() when the receive operation fails due to the queue already being empty. The pxQueue parameter passes the handle of the target queue into the macro.
- *traceTASK_DELAY_UNTIL( xTimeToWake )* Called from within `xTaskDelayUntil()` immediately before the calling task enters the Blocked state.
- *traceTASK_DELAY()* Called from within `vTaskDelay()` immediately before the calling task enters the `Blocked` state.
#### Defining Trace Hook Macros
Each trace macro has a `default empty` definition. The default definition can be **overridden** by providing a new macro definition in `FreeRTOSConfig.h`. If trace macro definitions become long or complex, then they can be implemented in a new header file that is then itself included from FreeRTOSConfig.h.

FreeRTOS maintains a `strict` data `hiding` policy. `Trace macros` allow user code to be added to the FreeRTOS source files, so the `data types` visible to the trace macros will be different to those visible to application code:
- Inside the `FreeRTOS/Source/tasks.c` source file, a *task handle* is a pointer to the data structure that describes a task (the task's `Task Control Block`, or **TCB**). Outside of the FreeRTOS/Source/tasks.c source file a *task handle* is a pointer to **void**.
- Inside the` FreeRTOS/Source/queue.c` source file, a `queue handle` is a pointer to the data structure that describes a **queue**. Outside of the FreeRTOS/Source/queue.c source file a `queue handle` is a pointer to **void**.
## Troubleshooting
### Introduction and Scope
This chapter highlights the most common issues encountered by users who are new to FreeRTOS.
#### Scope
- [ ] incorrect interrupt priority assignment
- [ ] stack overflow
- [ ] inappropriate use of printf()
### Interrupt Priorities
If the FreeRTOS port in use supports *interrupt nesting*, and the service routine for an interrupt makes use of the `FreeRTOS API`, then it is essential the **interrupt's priority** is set at or *below* `configMAX_SYSCALL_INTERRUPT_PRIORITY`,Failure to do this will result in ineffective **critical sections**, which in turn will result in `intermittent failures`.

Take particular care if running FreeRTOS on a processor where:
- Interrupt priorities default to having the *highest possible priority*, which is the case on some `ARM Cortex` processors, and possibly others. On such processors, the priority of an interrupt that uses the FreeRTOS API cannot be left `uninitialized`.
- `Numerically high priority` numbers represent *logically low interrupt priorities*, which may seem counterintuitive, and therefore cause confusion. Again this is the case on `ARM Cortex` processors, and possibly others.
- The bits that define the priority of an interrupt can be split between bits that define a `pre-emption priority`, and bits that define a `sub-priority`. Ensure *all the bits* are assigned to specifying a `pre-emption priority`, so that sub-priorities are not used.
### Stack Overflow
#### uxTaskGetStackHighWaterMark(
Each task maintains its *own stack*, the total size of which is specified when the task is created. `uxTaskGetStackHighWaterMark()` is used to query how close a task has come to overflowing the stack space allocated to it. This value is called the stack *'high water mark'*.
```c
/**
 * @brief Returns the minimum amount of free stack space that has been available to a task since it started.
 *
 * @param xTask The handle of the task whose stack high water mark is being queried.
 *               If a task is querying its own high water mark, it can pass NULL.
 *
 * @return The minimum amount of remaining stack space (in words) that has been available
 *         since the task started executing. This is the amount of stack that remains
 *         unused when stack usage is at its greatest (or deepest) value. The closer the
 *         high water mark is to zero, the closer the task has come to overflowing its stack.
 *
 * uxTaskGetStackHighWaterMark() parameters and return value:
 * - xTask: The handle of the task (the subject task). Handles to tasks can be obtained
 *   from the pxCreatedTask parameter of the xTaskCreate() API function. A task can query
 *   its own stack high water mark by passing NULL in place of a valid task handle.
 * - Return value: This function returns the minimum amount of free stack space that has
 *                 been available to the task since it started. The value indicates how
 *                 close the task has come to overflowing its stack; the closer to zero,
 *                 the closer to a stack overflow.
 */
UBaseType_t uxTaskGetStackHighWaterMark(TaskHandle_t xTask);
```
#### Run Time Stack Checking—Overview
FreeRTOS includes `three optional` run time stack checking mechanisms. These are controlled by the `configCHECK_FOR_STACK_OVERFLOW` compile time configuration constant within `FreeRTOSConfig.h`. Both methods increase the time it takes to perform a *context switch*.

The `stack overflow hook` (or stack overflow callback) is a function that is called by the *kernel* when it detects a `stack overflow`.
- Provide the implementation of the hook function, using the exact function name and `prototype`
```c
void vApplicationStackOverflowHook( TaskHandle_t *pxTask, signed char *pcTaskName );
```

The stack overflow hook is provided to make `trapping and debugging` stack errors easier, but there is `no real way` to *recover* from a stack overflow when it occurs. The function's parameters pass the handle and name of the task that has `overflowed` its stack into the hook function.

> The `stack overflow hook` gets called from the **context of an interrupt**.

Some microcontrollers generate a `fault exception` when they detect an incorrect memory access, and it is possible for a fault to be `triggered` before the kernel has a chance to call the stack overflow hook function.
#### Run Time Stack Checking—Method 1
> `configCHECK_FOR_STACK_OVERFLOW` is set to `1`.

A task's `entire execution context` is saved onto its stack each time it gets *swapped out*. It is likely that this will be the time at which stack usage reaches its **peak**.

The kernel checks that **the stack pointer** remains within the `valid stack space` after the context has `been saved`. The stack overflow hook is called if the stack pointer is found to be *outside* its valid range.

Method 1 is *quick* to execute, but can `miss` stack overflows that occur `between context switches`.

#### Run Time Stack Checking—Method 2
> `configCHECK_FOR_STACK_OVERFLOW` is set to `2`.

When a task is created, its stack is filled with a *known pattern*.

Method 2 tests the `last valid 20 bytes` of the task stack space to verify that this pattern has not been *overwritten*. The stack overflow hook function is called if any of the 20 bytes have changed from their expected values.

Method 2 is not `as quick` to execute as method 1, but is still relatively fast, as `only 20 bytes` are tested. Most likely, it will catch ==all stack overflows==; however, it is possible (but highly improbable) that some overflows will be missed.
#### Run Time Stack Checking—Method 3
> `configCHECK_FOR_STACK_OVERFLOW` is set to `3`.

This method is available only for `selected ports`. When available, this method enables **ISR stack checking**. When an ISR stack overflow is detected, an *assert is triggered*. Note that the `stack overflow hook` function is `not called` in this case because it is specific to a task stack and not the ISR stack.

### Use of printf() and sprintf()
Logging via `printf()` is a common source of error.

Many cross compiler vendors will provide a `printf()` implementation that is suitable for use in small embedded systems.
Even when that is the case, the implementation may **not be thread safe**, probably won't be suitable for use inside an **interrupt service routine**, and depending on where the `output is directed`, take a relatively **long time to execute**.

A generic printf() implementation is used instead, as:
- Just including a call to `printf()` or `sprintf()` can *massively increase* the **size** of `the application's executable`.
- `printf()` and `sprintf()` may call *malloc()*, which might be `invalid` if a memory allocation scheme `other than` `heap_3` is in use.
- `printf()` and `sprintf()` may require a stack that is many times *bigger than* would otherwise be required.
#### Printf-stdarg.c
Many of the `FreeRTOS demonstration projects` use a file called `printf-stdarg.c`, which provides a *minimal and stack-efficient* implementation of sprintf() that can be used in place of the standard library version. In most cases, this will permit a much smaller stack to be allocated to each task that calls sprintf() and related functions.

Note that `not all` copies of printf-stdarg.c included in the FreeRTOS download implement snprintf(). Copies that do not implement` snprintf()` simply *ignore* the buffer size parameter, as they map directly to `sprintf()`.
### Other Common Sources of Error
#### Symptom: Adding a simple task to a demo causes the demo to crash
The `idle task`, and possibly also the `RTOS daemon task`, are created automatically when `vTaskStartScheduler()` is called.
`vTaskStartScheduler()` will return only if there is *not enough* heap memory remaining for these tasks to be created. Including a null loop `for(;;);` ]after the call to `vTaskStartScheduler()` can make this error easier to *debug*.
#### Symptom: Using an API function within an interrupt causes the application to crash
In FreeRTOS ports that support `interrupt nesting`, do not use `any API functions` in an interrupt that has been assigned an interrupt priority above `configMAX_SYSCALL_INTERRUPT_PRIORITY`.
#### Symptom: The scheduler crashes when attempting to start the first task
Ensure the `FreeRTOS interrupt handlers` have been installed.

Some processors must be in a `privileged mode` before the scheduler can be started. The easiest way to achieve this is to place the processor into a privileged mode within the C startup code, `before main()` is called.
#### Symptom: Interrupts are unexpectedly left disabled, or critical sections do not nest correctly
If a FreeRTOS API function is called *before* the scheduler has been started then interrupts will deliberately be `left disabled`, and `not re-enabled again `until the first task starts to execute.

Do not alter the microcontroller interrupt enable bits or priority flags using any method other than calls to `taskENTER_CRITICAL()` and `taskEXIT_CRITICAL()`.

#### Symptom: The application crashes even before the scheduler is started
An `interrupt service routine` that could potentially cause a `context switch` must not be permitted to execute before the `scheduler` has been started.

A *context switch* cannot occur until after the scheduler has started.

#### Symptom: Calling API functions while the scheduler is suspended, or from inside a critical section, causes the application to crash
Do not call `API functions` while the scheduler is **suspended**, or from inside a **critical section**.
### Additional Debugging Steps
- Define `configASSERT()`, enable `malloc failed checking` and `stack overflow checking` in the application's FreeRTOSConfig file.
- Check the `return values` of the FreeRTOS APIs to make sure those were successful.
- Check that the secheduler related configuration, like `configUSE_TIME_SLICING`, and `configUSE_PREEMPTION` are set correctly as per the application requirements.
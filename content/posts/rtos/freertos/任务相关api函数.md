+++
title = "任务相关API函数"
date = 2025-07-10
description = "整理 任务相关API函数 相关笔记。"

[taxonomies]
tags = ["freertos", "rtos"]
+++
# 1.任务相关API函数

| 函数                                   | 描述                           |
| ------------------------------------ | ---------------------------- |
| uxTaskPriorityGet()                  | 查询某个任务的优先级                   |
| vTaskPrioritySet()                   | 改变某个任务的优先级                   |
| uxTaskGetSystemState()               | 获取系统中任务状态                    |
| vTaskGetInfo()                       | 获取某个任务信息                     |
| xTaskGetApplicationTaskTag()         | 获取某个任务的标签值                   |
| xTaskGetCurrentTaskHandle()          | 获取当前正在运行任务的任务句柄              |
| xTaskGetHandle()                     | 根据任务名字查找某个任务的句柄              |
| xTaskGetIdleTaskHandle()             | 获取空闲任务的句柄                    |
| uxTaskGetStackHighWaterMark()        | 获取任务的堆栈的历史剩余最小值，也叫高水位线       |
| eTaskGetState()                      | 获取某个任务的状态，该状态时 eTaskState 类型 |
| pcTaskGetName()                      | 获取某个任务的任务名字                  |
| xTaskGetTickCount()                  | 获取系统时间计数器值                   |
| xTaskGetTickCountFromISR()           | 在中断服务函数中获取时间计数器值             |
| xTaskGetSchedulerState()             | 获取任务调度器的状态，开启或未开启            |
| uxTaskGetNumberOfTask()              | 获取当前系统中存在的任务数量               |
| vTaskList()                          | 以表格的形式输出当前系统中所有任务的详细信息       |
| vTaskGetRunTimeStats()               | 获取每个任务的运行时间                  |
| vTaskSetApplicationTaskTag()         | 设置任务标签值                      |
| vTaskSetThreadLocalStoragePointer()  | 设置线程本地存储指针                   |
| pvTaskGetThreadLocalStoragePointer() | 获取线程本地存储指针                   |

# 2.相关API函数详解
- `uxTaskPriorityGet()`  :  查询某个任务的优先级

```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_uxTaskPriorityGet 			 必须置为1
/*******************************函数原型*******************************/
函数原型： UBaseType_t uxTaskPriorityGet(TaskHandle_t xTask)

```
**传 入 值**：`xTask` 要查找的任务的任务句柄
**返 回 值**：获取到的对应的任务的优先级

- `vTaskPrioritySet()`  :  改变某个任务的优先级
```c
/*****************************相关宏的配置****************************/
#define INCLUDE_vTaskPrioritySet 			 必须置为1
/*******************************函数原型******************************/
函数原型：void vTaskPrioritySet(TaskHandle_t xTask,UBaseType_t uxNewPriority)
```
**传 入 值**：*xTask* 要更改优先级的任务的任务句柄
			 *uxNewPriority* 任务要使用的新的优先级

- `uxTaskGetSystemState()` ：获取系统中所有任务的任务状态，每个任务的状态信息保存在一个 TaskStatus_t 类型的结构体里，这个结构体包含了任务句柄、任务名称、堆栈、优先级等信息
```c
/*****************************相关宏的配置*****************************/
#define configUSE_TRACE_FACILITY 			 必须置为1
/*******************************函数原型*******************************/
函数原型： UBaseType_t uxTaskGetSystemState(TaskStatus_t *const pxTaskStatusArray,
										  const UBaseType_t uxArraySize,
										  uint32_t *const pulTotalRunTime)
```
**传 入 值**：`pxTaskStatusArray` 指向TaskStatus_t结构体类型的数组首地址
		 `uxArraySize` 	   保存任务状态数组的大小
		 `pulTotalRunTime`   若开启了系统运行时间统计(`configGENERATE_RUN_TIME_STATS  1`)，则用来保存系统总的运行时间
			 
**返 回 值**：统计到的任务状态的个数，也就是填写到数组`pxTaskStatusArray`中的个数
```c
/*****TaskStatus_t结构体*****/
typedef struct xTASK_STATUS{
	TaskHandle_t	xHandle;	//任务句柄
	const char*		pcTaskName;	//任务名字
	UBaseType_t		xTaskNumber;	//任务编号
	eTaskState		eCurrentState;	//当前任务状态
	UBaseType_t		uxCurrentPriority;	//任务当前的优先级
	UBaseType_t		uxBasePriority;		//任务基础优先级
	uint32_t		ulRunTimeCounter;	//任务运行的总时间
	StackType_t*	pxStackBase;	//堆栈基地址
	uint16_t		usStackHighWaterMark;//任务创建依赖任务堆栈剩余的最小值
}TaskStatus_t;
```

- `vTaskGetInfo()` ：获取某个指定的单个任务的任务信息
```c
/*****************************相关宏的配置*****************************/
#define configUSE_TRACE_FACILITY 			 必须置为1
/*******************************函数原型*******************************/
函数原型：void vTaskGetInfo(TaskHandle_t	xTask,
						   TaskStatus_t* pxTaskStatus,
						   BaseType_t xGetFreeStackSpace,
						   eTaskState  eState)
```
**传 入 值**：`xTask` 要查找的任务的任务句柄
			 `pxTaskStatus` 指向类型为TaskStatus_t结构体变量
			 `xGetFreeStackSpace` 堆栈剩余的历史最小值
			 `eState` 保存任务运行状态
			 
 - `xTaskGetApplicationTaskTag()` ：获取某个任务的标签值
```c
 /*****************************相关宏的配置*****************************/
#define configUSE_APPLICATION_TASK_TAG	 必须置为1
/*******************************函数原型*******************************/
函数原型：void xTaskGetApplicationTaskTag(TaskHandle_t xTask)
```
**传 入 值**：`xTask` 要获取标签值的任务的任务句柄，若为NULL表示获取当前任务的标签值
**返 回 值**：任务的标签值

- `xTaskGetCurrentTaskHandle()` ：获取当前正在运行任务的任务句柄，其实获取到的就是任务控制块
```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_xTaskGetCurrentTaskHandle	必须置为1
/*******************************函数原型*******************************/
函数原型： TaskHandle_t xTaskGetCurrentTaskHandle(void)
```
**返 回 值**：当前任务的任务句柄

- `xTaskGetHandle()` ：根据任务名字查找某个任务的句柄
```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_xTaskGetHandle	必须置为1
/*******************************函数原型*******************************/
函数原型： TaskHandle_t xTaskGetHandle(const char* pcNameToQuery)
```
**传 入 值**：`pcNameToQuery` 任务名
**返 回 值**：任务名所对应的任务句柄；返回NULL表示没有对应的任务

- `xTaskGetIdleTaskHandle()`：获取空闲任务的句柄
```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_xTaskGetIdleTaskHandle	必须置为1
/*******************************函数原型*******************************/
函数原型： TaskHandle_t xTaskGetIdleTaskHandle(void)
```
**返 回 值**：空闲任务的任务句柄

- `uxTaskGetStackHighWaterMark()` ：每个任务在创建的时候就确定了堆栈大小，此函数用于检查任务从创建好到现在的历史剩余最小值，这个值越小说明任务堆栈溢出的可能性就越大
```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_uxTaskGetStackHighWaterMark	必须置为1
/*******************************函数原型*******************************/
函数原型： UBaseType_t uxTaskGetStackHighWaterMark(TaskHandle_t xTask)
```
**传 入 值**：`xTask` 要查询的任务的任务句柄，若为NULL表示查询自身任务的高水位线
**返 回 值**：任务堆栈的高水位线值，即堆栈的历史剩余最小值

- `eTaskGetState()` ：获取某个任务的状态，比如：运行态、阻塞态、挂起态、就绪态等
```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_eTaskGetState	必须置为1
/*******************************函数原型*******************************/
函数原型：eTaskState eTaskGetState(TaskHandle_t xTask)
```
**传 入 值**：`xTask` 要查询的任务的任务句柄
**返 回 值**：返回`eTaskState`枚举类型值
```c
typedef enum 
{ 
	eRunning = 0, /* 任务正在查询自身的状态，所以必须是运行状态 */ 
	eReady, /* 就绪状态 */ 
	eBlocked, /* 阻塞状态 */ 
	eSuspended, /* 处于挂起状态，或者处于阻塞状态但超时时间为无限 */ 
	eDeleted, /* 被查询的任务已被删除，但其任务控制块 (TCB) 尚未被释放 */ 
	eInvalid /* 无效状态 */ 
} eTaskState;
```

- `pcTaskGetName()` ：根据任务句柄来获取该任务的任务名字
```c
/*******************************函数原型*******************************/
函数原型：char* pcTaskGetName(TaskHandle_t xTaskToQuery)
```
**传 入 值**：`xTaskToQuery` 要查询的任务的任务句柄，若为NULL表示查询自身任务名
**返 回 值**：返回任务所对应的任务名

- `xTaskGetTickCount()` ：用于查询任务调度器从启动到现在时间计数器 xTickCount 的值。xTickCount 是系统的时钟节拍值，并不是真实的时间值。每个滴答定时器中断 xTickCount 就会加一，中断周期取决于系统时钟节拍数
```c
/*******************************函数原型*******************************/
函数原型： TickType_t xTaskGetTickCount(void)
```
  **返 回 值**：时间计数器`xTickCount`的值

- `xTaskGetTickCountFromISR()` ：在中断服务函数中获取时间计数器值
```c
/*******************************函数原型*******************************/
函数原型： TickType_t xTaskGetTickCountFromISR(void)
```
**返 回 值**：时间计数器xTickCount的值

- `xTaskGetSchedulerState()` ：获取任务调度器的状态，开启、关闭还是挂起
```c
/*****************************相关宏的配置*****************************/
#define INCLUDE_xTaskGetSchedulerState	必须置为1
/*******************************函数原型*******************************/
函数原型： BaseType_t xTaskGetSchedulerState(void)
```
**返 回 值**：   `taskCHEDULER_NOT_STARTED` 调度器未启动
			`taskCHEDULER_RUNNING` 调度器正在运行
			`taskCHEDULER_SUSPENDED` 调度器挂起

- `uxTaskGetNumberOfTask()` ：获取当前系统中存在的任务数量
```c
/*******************************函数原型*******************************/
函数原型： UBaseType_t uxTaskGetNumberOfTask(void)
```
**返 回 值**：当前系统中存在的任务数量，此值包含各种状态下的任务数

- `vTaskList()` ：以表格的形式输出当前系统中所有任务的详细信息
```c
/*******************************函数原型*******************************/
函数原型：void vTaskList(char *pcWriteBuffer)
```
**传 入 值**：pcWriteBuffer 保存任务状态信息表的存储区

- `vTaskGetRunTimeStats()` ：获取每个任务的运行时间
```c
/*****************************相关宏的配置*****************************/
#define configUSE_TRACE_FACILITY                  必须置为1
#define configGENERATE_RUN_TIME_STATS 			  必须置为1 
#define configUSE_STATS_FORMATTING_FUNCTIONS	  必须置为1
/*******************************函数原型*******************************/
函数原型：void vTaskGetRunTimeStats(char *pcWriteBuffer)
```
**传 入 值**：`pcWriteBuffer` 保存任务时间信息的存储区

- `vTaskSetApplicationTaskTag()` ：用于设置某个任务的标签值
```c
/*****************************相关宏的配置*****************************/
#define configUSE_APPLICATION_TASK_TAG	 必须置为1
/*******************************函数原型*******************************/
函数原型：void vTaskSetApplicationTaskTag(TaskHandle_t xTask，
										TaskHookFunction_t pxHookFunction)
```
**传 入 值**：`xTask` 要要设置标签值的任务的任务句柄
			 `pxHookFunction` 要设置的标签值

- `vTaskSetThreadLocalStoragePointer()` ：设置线程本地存储指针的值，每个任务都有自已的指针数组作为线程本地存储，使用这些线程本地存储可以用来在任务控制块中存储一些只属于任务自已的应用信息
```c
/*****************************相关宏的配置*****************************/
#define configNUM_THREAD_LOCAL_STORAGE_POINTERS	 本地存储指针数组的大小
/*******************************函数原型*******************************/
函数原型：void vTaskSetThreadLocalStoragePointer(TaskHandle_t xTaskToSet,
												BaseType_t xIndex,
												void* pvValue)
```
**传 入 值**：**xTaskToSet** 要设置线程本地存储指针的任务的任务句柄
			 **xIndex** 要设置的线程本地存储指针数组的索引
			 **pvValue** 要存储的值

- `pvTaskGetThreadLocalStoragePointer()` ：获取线程本地存储指针
```c
/*****************************相关宏的配置*****************************/
#define configNUM_THREAD_LOCAL_STORAGE_POINTERS	 本地存储指针数组的大小
/*******************************函数原型*******************************/
函数原型：void *pvTaskGetThreadLocalStoragePointer(TaskHandle_t xTaskToQuery,
                                 				  BaseType_t xIndex )
```
**传 入 值**：`xTaskToQuery` 要获取线程本地存储指针的任务的任务句柄
			 `xIndex` 要获取的线程本地存储指针数组的索引

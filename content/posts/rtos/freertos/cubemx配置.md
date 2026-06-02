+++
title = "cubemx配置"
date = 2025-12-10
description = "整理 cubemx配置 相关笔记。"

[taxonomies]
tags = ["freertos", "rtos"]
+++
# 1.config parameters
## 1.1.内核设置
- **USE_PREEMPTION**:
	- `Enable`:抢占式任务调度
	- `Disable`:合作式任务调度
- **TICK_RATE_HZ**:`1-1000`系统滴答时钟的时钟频率
- **MINIMAL_STACK_SIZ**:`64-3840字`系统任务栈空间的最小大小
- **MAX_TASK_NAME_LEN**:`12-255`任务名最大字符串长度
- **IDLE_SHOULD_YIELD**:空闲任务是否对同优先级任务主动让出cpu使用权
- **QUEUE_REGISTRY_SIZE**:可注册的队列和信号量的最大数量
- **USE_APPLICATION_TASK_TAG**:是否使用应用程序的任务标签
- **USE_TICKLESS_IDLE**:是否在空闲时进入低功耗模式
- **USE_TASK_NOTIFICATIONS**:是否使用任务通知功能
- **RECORD_STACK_HIGH_ADDRESS**:是否使用 栈的起始地址保存到每个任务的任务控制块中
- **TOTAL_HEAP_SIZE**:总的堆大小
## 1.2.**五个钩子函数**:
- **USE_IDLE_HOOK**
- **USE_TICK_HOOK**
- **USE_MALLOC_FAILED_HOOK**
- **USE_DAEMON_TASK_STARTUP_HOOK**
- **CHECK_FOR_STACK_OVERFLOW**
## 1.3.运行时间和任务状态收集
- **GENERATE_RUN_TIME_STATS**:是否启用任务运行时间启用功能
- **USE_TRACE_FACILITY**:是否启用一些用于可视化和追踪调试的功能
- **USE_STATS_FORMATTING_FUNCTIONS**:是否编译 vTaskList()和vTaskGetRunTimeStats()
## 1.4.软件定时器
- **TIMER_TASK_PRIORITY**：定时器任务的优先级
- **TIMER_QUEUE_LENGTH**：定时器指令队列的长度
- **TIMER_TASK_STACK_DEPTH**：定时器任务的栈空间大小
- 

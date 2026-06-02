+++
title = "静态创建task（手动挡）"
date = 2025-07-10
description = "整理 静态创建task（手动挡） 相关笔记。"

[taxonomies]
tags = ["freertos", "rtos"]
+++
# 示例代码
```c
//任务优先级
#define TASK_PRIORITY 1

//任务堆栈大小
#define TASK_STACK_SIZE 128
//任务堆栈
StackType_t task_stack[TASK_STACK_SIZE];
//任务控制块
StaticTask_t task_control_block;
//任务句柄
TaskHandle_t task_handle;
//任务函数
void start_task(void* pvParameters);
task_handle = xTaskCreateStatic(
        start_task,
        "start_task",
        TASK_STACK_SIZE,
        NULL,
        TASK_PRIORITY,
        task_stack,
        &task_control_block);
    vTaskStartScheduler();


void start_task(void* pvParameters)
{
    taskENTER_CRITICAL();//进入临界区

    //任务初始化
    task1_handle = xTaskCreateStatic(
        task1,
        "task1",
        TASK_STACK_SIZE,
        NULL,
        TASK1_PRIORITY,
        task1_stack,
        &task1_control_block);

    task2_handle = xTaskCreateStatic(
        task2,
        "task2",
        TASK_STACK_SIZE,
        NULL,
        TASK2_PRIORITY,
        task2_stack,
        &task2_control_block);
    vTaskDelete(NULL);

    taskEXIT_CRITICAL();//退出临界区

}
```

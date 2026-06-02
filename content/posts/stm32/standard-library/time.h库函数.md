+++
title = "time.h库函数"
date = 2026-05-19
description = "整理 time.h库函数 相关笔记。"

[taxonomies]
tags = ["stm32", "standard-library"]
+++
|函数|作用|
|---|---|
|time_t time(time_t*);|获取系统时钟|
|struct tm* gmtime(const time_t*);|秒计数器转换为日期时间（格林尼治时间）|
|struct tm* localtime(const time_t*);|秒计数器转换为日期时间（当地时间）|
|time_t mktime(struct tm*);|日期时间转换为秒计数器（当地时间）|
|char* ctime(const time_t*);|秒计数器转换为字符串（默认格式）|
|char* asctime(const struct tm*);|日期时间转换为字符串（默认格式）|
|size_t strftime(char*, size_t, const char*, const struct tm*);|日期时间转换为字符串（自定义格式）|


```c
struct tm {
    int tm_sec;   /* 秒 - 取值区间为[0,59] */
    int tm_min;   /* 分 - 取值区间为[0,59] */
    int tm_hour;  /* 时 - 取值区间为[0,23] */
    int tm_mday;  /* 一个月中的日期 - 取值区间为[1,31] */
    int tm_mon;   /* 月份(从一月开始，0代表一月)- 取值区间为[0,11] */
    int tm_year;  /* 年份，其值等于实际年份减去1900 */
    int tm_wday;  /* 星期–取值区间为[0,6]，其中0代表星期天，1代表星期一，以此类推 */
    int tm_yday;  /* 从每年的1月1日开始的天数 – 取值区间为[0,365]，其中0代表1月1日，1代表1月2日，以此类推 */
    int tm_isdst; /* 夏令时标识符，实行夏令时的时候，tm_isdst为正。不实行夏令时的进候，tm_isdst为0；不了解情况时，tm_isdst()为负。*/
};
```



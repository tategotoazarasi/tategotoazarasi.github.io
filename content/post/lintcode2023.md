---
title: "Lintcode 2023. 计算 2018 年 01 月 13 日到课程创建时间的天数"
date: 2022-01-02T09:24:13+08:00
draft: false
tags: ["lintcode", "入门", "数据库", "时间函数"]
math: true
---

请编写 SQL 语句，查询 `courses` 表，计算从 2018 年 01 月 13 日到创建课程时间（`created at`）相差天数，结果列名请以 `date_diff` 显示。

<!--more-->

表定义：courses (课程表)

| 列名            | 类型           | 注释     |
| ------------- | ------------ | ------ |
| id            | int unsigned | 主键     |
| name          | varchar      | 课程名称   |
| student count | int          | 学生总数   |
| created at    | date         | 创建课程时间 |
| teacher id    | int          | 讲师 id  |

> - 如果课程创建时间早于 2018 年 01 月 13 日，所计算返回的天数是负值
> 
> - 如果有课程记录的创建时间数据为空，返回 NULL

## 样例

样例一：

表内容：courses (课程表)

| **id** | **name**                | **student_count** | **created_at** | **teacher_id** |
| ------ | ----------------------- | ----------------- | -------------- | -------------- |
| 1      | Senior Algorithm        | 880               | 2018-01-13     | 4              |
| 2      | System Design           | 1350              | 2020-07-18     | 3              |
| 3      | Django                  | 780               | 2020-02-29     | 3              |
| 4      | Web                     | 340               | 2020-04-22     | 4              |
| 5      | Big Data                | 700               | 2020-09-11     | 1              |
| 6      | Artificial Intelligence | 1660              | 2018-05-13     | 3              |
| 7      | Java P6+                | 780               | 2019-01-19     | 3              |
| 8      | Data Analysis           | 500               | 2019-07-12     | 1              |
| 10     | Object Oriented Design  | 300               | 2020-08-08     | 4              |
| 12     | Dynamic Programming     | 2000              | 2018-08-18     | 1              |

在运行你的 SQL 语句之后，表应返回：

| **date_diff** |
| ------------- |
| 0             |
| 917           |
| 777           |
| 830           |
| 972           |
| 120           |
| 371           |
| 545           |
| 938           |
| 217           |

样例二：

表内容：courses (课程表)

| **id** | **name**         | **student_count** | **created_at** | **teacher_id** |
| ------ | ---------------- | ----------------- | -------------- | -------------- |
| 1      | Senior Algorithm | 880               | null           | 4              |
| 2      | System Design    | 1350              | null           | 3              |

在运行你的 SQL 语句之后，表应返回：

**date_diff**

> 因为样例中是空日期数据，所以这里只展示了标题，没有数据。

```sql
select DATEDIFF(created_at, DATE("2018.1.31")) as data_diff
from courses;
```

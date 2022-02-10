---
title: "LeetCode 197. Rising Temperature"
date: 2022-02-10T11:05:17+08:00
draft: false
tags: ["leetcode", "简单", "数据库"]
math: true
---

```sql
Create table If Not Exists Weather
(
    id          int,
    recordDate  date,
    temperature int
);
Truncate table Weather;
insert into Weather (id, recordDate, temperature)
values ('1', '2015-01-01', '10');
insert into Weather (id, recordDate, temperature)
values ('2', '2015-01-02', '25');
insert into Weather (id, recordDate, temperature)
values ('3', '2015-01-03', '20');
insert into Weather (id, recordDate, temperature)
values ('4', '2015-01-04', '30');
```

Table: `Weather`

| Column Name | Type |
| ----------- | ---- |
| id          | int  |
| recordDate  | date |
| temperature | int  |

id is the primary key for this table.
This table contains information about the temperature on a certain day.

Write an SQL query to find all dates' `Id` with higher temperatures compared to its previous dates (yesterday).

Return the result table in **any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Weather table:

| id  | recordDate | temperature |
| --- | ---------- | ----------- |
| 1   | 2015-01-01 | 10          |
| 2   | 2015-01-02 | 25          |
| 3   | 2015-01-03 | 20          |
| 4   | 2015-01-04 | 30          |

**Output:**

| id  |
| --- |
| 2   |
| 4   |

**Explanation:**
In 2015-01-02, the temperature was higher than the previous day (10 -> 25).
In 2015-01-04, the temperature was higher than the previous day (20 -> 30).

## Solution

MySQL 使用 [DATEDIFF](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_datediff) 来比较两个日期类型的值。

因此，我们可以通过将 **weather** 与自身相结合，并使用 `DATEDIFF()` 函数。

```sql
SELECT Weather.id AS 'Id'
FROM Weather
         JOIN
     Weather w ON DATEDIFF(Weather.recordDate, w.recordDate) = 1
         AND Weather.Temperature > w.Temperature
;
```

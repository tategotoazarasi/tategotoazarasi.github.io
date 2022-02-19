---
title: "LeetCode 1179. Reformat Department Table"
date: 2022-02-19T14:31:05+08:00
draft: false
tags: ["leetcode", "简单", "数据库"]
math: true
---

Table: `Department`

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| revenue     | int     |
| month       | varchar |

(id, month) is the primary key of this table.
The table has information about the revenue of each department per month.
The month has values in ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"].

Write an SQL query to reformat the table such that there is a department id column and a revenue column **for each month**.

Return the result table in **any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Department table:

| id  | revenue | month |
| --- | ------- | ----- |
| 1   | 8000    | Jan   |
| 2   | 9000    | Jan   |
| 3   | 10000   | Feb   |
| 1   | 7000    | Feb   |
| 1   | 6000    | Mar   |

**Output:**

| id  | Jan_Revenue | Feb_Revenue | Mar_Revenue | ... | Dec_Revenue |
| --- | ----------- | ----------- | ----------- | --- | ----------- |
| 1   | 8000        | 7000        | 6000        | ... | null        |
| 2   | 9000        | null        | null        | ... | null        |
| 3   | null        | 10000       | null        | ... | null        |

**Explanation:** The revenue from Apr to Dec is null.
Note that the result table has 13 columns (1 for the department id + 12 for the months).

## Solution

#### Solution 1

```sql
SELECT id,
    SUM(
        CASE
            WHEN month = 'Jan' THEN revenue
        END
    ) AS Jan_Revenue,
    SUM(
        CASE
            WHEN month = 'Feb' THEN revenue
        END
    ) AS Feb_Revenue,
    SUM(
        CASE
            WHEN month = 'Mar' THEN revenue
        END
    ) AS Mar_Revenue,
    SUM(
        CASE
            WHEN month = 'Apr' THEN revenue
        END
    ) AS Apr_Revenue,
    SUM(
        CASE
            WHEN month = 'May' THEN revenue
        END
    ) AS May_Revenue,
    SUM(
        CASE
            WHEN month = 'Jun' THEN revenue
        END
    ) AS Jun_Revenue,
    SUM(
        CASE
            WHEN month = 'Jul' THEN revenue
        END
    ) AS Jul_Revenue,
    SUM(
        CASE
            WHEN month = 'Aug' THEN revenue
        END
    ) AS Aug_Revenue,
    SUM(
        CASE
            WHEN month = 'Sep' THEN revenue
        END
    ) AS Sep_Revenue,
    SUM(
        CASE
            WHEN month = 'Oct' THEN revenue
        END
    ) AS Oct_Revenue,
    SUM(
        CASE
            WHEN month = 'Nov' THEN revenue
        END
    ) AS Nov_Revenue,
    SUM(
        CASE
            WHEN month = 'Dec' THEN revenue
        END
    ) AS Dec_Revenue
FROM Department
GROUP BY id
ORDER BY id;
```

#### Solution 2

```sql
SELECT DISTINCT d0.id,
    d1.revenue as Jan_Revenue,
    d2.revenue as Feb_Revenue,
    d3.revenue as Mar_Revenue,
    d4.revenue as Apr_Revenue,
    d5.revenue as May_Revenue,
    d6.revenue as Jun_Revenue,
    d7.revenue as Jul_Revenue,
    d8.revenue as Aug_Revenue,
    d9.revenue as Sep_Revenue,
    d10.revenue as Oct_Revenue,
    d11.revenue as Nov_Revenue,
    d12.revenue as Dec_Revenue
FROM Department d0
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Jan'
    ) d1 ON d0.id = d1.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Feb'
    ) d2 ON d0.id = d2.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Mar'
    ) d3 ON d0.id = d3.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Apr'
    ) d4 ON d0.id = d4.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'May'
    ) d5 ON d0.id = d5.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Jun'
    ) d6 ON d0.id = d6.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Jul'
    ) d7 ON d0.id = d7.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Aug'
    ) d8 ON d0.id = d8.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Sep'
    ) d9 ON d0.id = d9.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Oct'
    ) d10 ON d0.id = d10.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Nov'
    ) d11 ON d0.id = d11.id
    LEFT JOIN (
        SELECT id,
            revenue
        FROM Department
        WHERE month = 'Dec'
    ) d12 ON d0.id = d12.id
ORDER BY d0.id;
```

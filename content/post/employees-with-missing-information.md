---
title: "LeetCode 1965. Employees With Missing Information"
date: 2022-04-07T12:05:28+08:00
draft: false
tags: ["leetcode", "简单", "数据库", "组合查询 & 指定选取"]
math: true
---

Table: `Employees`

| Column Name | Type    |
| ----------- | ------- |
| employee_id | int     |
| name        | varchar |

employee_id is the primary key for this table.
Each row of this table indicates the name of the employee whose ID is employee_id.

Table: `Salaries`

| Column Name | Type |
| ----------- | ---- |
| employee_id | int  |
| salary      | int  |

employee_id is the primary key for this table.
Each row of this table indicates the salary of the employee whose ID is employee_id.

Write an SQL query to report the IDs of all the employees with **missing information**. The information of an employee is missing if:

- The employee's **name** is missing, or
- The employee's **salary** is missing.

Return the result table ordered by `employee_id` **in ascending order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Employees table:

| employee_id | name     |
| ----------- | -------- |
| 2           | Crew     |
| 4           | Haven    |
| 5           | Kristian |

Salaries table:

| employee_id | salary |
| ----------- | ------ |
| 5           | 76071  |
| 1           | 22517  |
| 4           | 63539  |

**Output:**

| employee_id |
| ----------- |
| 1           |
| 2           |

**Explanation:**
Employees 1, 2, 4, and 5 are working at this company.
The name of employee 1 is missing.
The salary of employee 2 is missing.

```sql
select employee_id
from (
         select employee_id
         from Employees
         union all
         select employee_id
         from Salaries
     ) as ans
group by employee_id
having count(employee_id) = 1
order by employee_id;
```

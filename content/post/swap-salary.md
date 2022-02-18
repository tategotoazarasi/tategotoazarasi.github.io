---
title: "LeetCode 627. Swap Salary"
date: 2022-02-18T10:09:38+08:00
draft: false
tags: ["leetcode", "简单", "数据库"]
math: true
---

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| name        | varchar |
| sex         | ENUM    |
| salary      | int     |

id is the primary key for this table.
The sex column is ENUM value of type ('m', 'f').
The table contains information about an employee.

Write an SQL query to swap all `'f'` and `'m'` values (i.e., change all `'f'` values to `'m'` and vice versa) with a **single update statement** and no intermediate temporary tables.

Note that you must write a single update statement, **do not** write any select statement for this problem.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Salary table:

| id  | name | sex | salary |
| --- | ---- | --- | ------ |
| 1   | A    | m   | 2500   |
| 2   | B    | f   | 1500   |
| 3   | C    | m   | 5500   |
| 4   | D    | f   | 500    |

**Output:**

| id  | name | sex | salary |
| --- | ---- | --- | ------ |
| 1   | A    | f   | 2500   |
| 2   | B    | m   | 1500   |
| 3   | C    | f   | 5500   |
| 4   | D    | m   | 500    |

**Explanation:**
(1, A) and (3, C) were changed from 'm' to 'f'.
(2, B) and (4, D) were changed from 'f' to 'm'.

## Solution

#### 方法一：使用 `UPDATE` 和 `CASE...WHEN`

**算法**

要想动态地将值设置成列，我们可以在使用 [`CASE...WHEN...`](https://dev.mysql.com/doc/refman/5.7/en/case.html) 流程控制语句的同时使用 [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/update.html) 语句。

```sql
UPDATE Salary
SET sex = CASE
        sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;
```

#### 方法一：使用 `UPDATE` 和 `IF`

```sql
UPDATE Salary
SET sex = IF(sex = 'm', 'f', 'm');
```

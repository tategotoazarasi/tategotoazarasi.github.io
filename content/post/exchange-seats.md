---
title: "LeetCode 626. Exchange Seats"
date: 2022-02-16T16:13:02+08:00
draft: false
tags: ["leetcode", "中等", "数据库"]
math: true
---

Table: `Seat`

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| name        | varchar |

id is the primary key column for this table.
Each row of this table indicates the name and the ID of a student.
id is a continuous increment.

Write an SQL query to swap the seat id of every two consecutive
students. If the number of students is odd, the id of the last student
is not swapped.

Return the result table ordered by `id` **in ascending order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Seat table:

| id  | student |
| --- | ------- |
| 1   | Abbot   |
| 2   | Doris   |
| 3   | Emerson |
| 4   | Green   |
| 5   | Jeames  |

**Output:**

| id  | student |
| --- | ------- |
| 1   | Doris   |
| 2   | Abbot   |
| 3   | Green   |
| 4   | Emerson |
| 5   | Jeames  |

**Explanation:**
Note that if the number of students is odd, there is no need to change the last one's seat.

## Solution

#### 方法一：使用 `CASE`【通过】

**算法**

对于所有座位 id 是奇数的学生，修改其 id 为 id+1，如果最后一个座位 id 也是奇数，则最后一个座位 id 不修改。对于所有座位 id 是偶数的学生，修改其 id 为 id-1。

首先查询座位的数量。

```sql
SELECT COUNT(*) AS counts
FROM Seat;
```

然后使用 `CASE` 条件和 `MOD` 函数修改每个学生的座位 id。

**MySQL**

```sql
SELECT (
        CASE
            WHEN MOD(id, 2) != 0
            AND counts != id THEN id + 1
            WHEN MOD(id, 2) != 0
            AND counts = id THEN id
            ELSE id - 1
        END
    ) AS id,
    student
FROM Seat,
    (
        SELECT COUNT(*) AS counts
        FROM Seat
    ) AS seat_counts
ORDER BY id ASC;
```

#### 方法二：使用位操作和 `COALESCE()`【通过】

**算法**

使用 `(id+1)^1-1` 计算交换后每个学生的座位 id。

```sql
SELECT id,
    (id + 1) ^ 1 - 1,
    student
FROM Seat;
```

| id  | (id+1)^1-1 | student |
| --- | ---------- | ------- |
| 1   | 2          | Abbot   |
| 2   | 1          | Doris   |
| 3   | 4          | Emerson |
| 4   | 3          | Green   |
| 5   | 6          | Jeames  |

然后连接原来的座位表和更新 id 后的座位表。

```sql
SELECT *
FROM Seat s1
    LEFT JOIN Seat s2 ON (s1.id + 1) ^ 1 - 1 = s2.id
ORDER BY s1.id;
```

| id  | student | id  | student |
| --- | ------- | --- | ------- |
| 1   | Abbot   | 2   | Doris   |
| 2   | Doris   | 1   | Abbot   |
| 3   | Emerson | 4   | Green   |
| 4   | Green   | 3   | Emerson |
| 5   | Jeames  |     |         |

> 注：前两列来自表 s1，后两列来自表 s2。

最后输出 s1.id 和 s2.student。但是 id=5 的学生，s1.student 正确，s2.student 为 NULL。因此使用 [`COALESCE()`](https://dev.mysql.com/doc/refman/5.7/en/comparison-operators.html#function_coalesce) 函数为最后一行记录生成正确的输出。

**MySQL**

```sql
SELECT s1.id,
    COALESCE(s2.student, s1.student) AS student
FROM Seat s1
    LEFT JOIN Seat s2 ON ((s1.id + 1) ^ 1) - 1 = s2.id
ORDER BY s1.id;
```

#### 方法四

```sql
SELECT b.id,
    a.student
FROM Seat a,
    Seat b
WHERE (
        a.id % 2 = 0
        AND b.id + 1 = a.id
    )
    OR (
        a.id % 2 = 1
        AND a.id + 1 = b.id
    )
    OR (
        a.id % 2 = 1
        AND a.id IN (
            SELECT MAX(id)
            FROM Seat
        )
        AND a.id = b.id
    )
ORDER BY b.id;
```

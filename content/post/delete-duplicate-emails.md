---
title: "LeetCode 196. Delete Duplicate Emails"
date: 2022-02-09T15:54:42+08:00
draft: false
tags: ["leetcode", "简单", "数据库"]
math: true
---

```sql
Create table If Not Exists Person
(
    Id    int,
    Email varchar(255)
);
Truncate table Person;
insert into Person (id, email)
values ('1', 'john@example.com');
insert into Person (id, email)
values ('2', 'bob@example.com');
insert into Person (id, email)
values ('3', 'john@example.com');
```

Table: `Person`

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| email       | varchar |

id is the primary key column for this table.
Each row of this table contains an email. The emails will not contain uppercase letters.

Write an SQL query to **delete** all the duplicate emails, keeping only one unique email with the smallest `id`. Note that you are supposed to write a `DELETE` statement and not a `SELECT` one.

Return the result table in **any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Person table:

| id  | email            |
| --- | ---------------- |
| 1   | john@example.com |
| 2   | bob@example.com  |
| 3   | john@example.com |

**Output:**

| id  | email            |
| --- | ---------------- |
| 1   | john@example.com |
| 2   | bob@example.com  |

**Explanation:** john@example.com is repeated two times. We keep the row with the smallest Id = 1.

## Solution

#### 方法一：使用 `DELETE` 和 `WHERE` 子句

**算法**

我们可以使用以下代码，将此表与它自身在*电子邮箱*列中连接起来。

```sql
SELECT p1.*
FROM Person p1,
     Person p2
WHERE p1.Email = p2.Email
;
```

然后我们需要找到其他记录中具有相同电子邮件地址的更大 ID。所以我们可以像这样给 `WHERE` 子句添加一个新的条件。

```sql
SELECT p1.*
FROM Person p1,
     Person p2
WHERE p1.Email = p2.Email
  AND p1.Id > p2.Id
;
```

因为我们已经得到了要删除的记录，所以我们最终可以将该语句更改为 `DELETE`。

```sql
DELETE p1
FROM Person p1,
     Person p2
WHERE p1.Email = p2.Email
  AND p1.Id > p2.Id;
```

#### 方法二

```sql
DELETE
FROM Person
WHERE Email IN (SELECT Email
                FROM (SELECT Email, COUNT(*) AS count
                      FROM Person
                      GROUP BY Email) b
                WHERE b.count >= 2)
  AND Id NOT IN (SELECT MIN(Id) AS id
                 FROM (SELECT *
                       FROM Person) c
                 GROUP BY c.Email);
```

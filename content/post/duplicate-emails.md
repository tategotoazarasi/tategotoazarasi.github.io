---
title: "LeetCode 182. Duplicate Emails"
date: 2022-02-05T10:57:27+08:00
draft: false
tags: ["leetcode", "简单", "数据库"]
math: true
---

```sql
Create table If Not Exists Person (id int, email varchar(255));
Truncate table Person;
insert into Person (id, email) values ('1', 'a@b.com');
insert into Person (id, email) values ('2', 'c@d.com');
insert into Person (id, email) values ('3', 'a@b.com');
```

Table: `Person`

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| email       | varchar |

id is the primary key column for this table.
Each row of this table contains an email. The emails will not contain uppercase letters.

Write an SQL query to report all the duplicate emails.

Return the result table in **any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Person table:

| id  | email   |
| --- | ------- |
| 1   | a@b.com |
| 2   | c@d.com |
| 3   | a@b.com |

**Output:**

| Email   |
| ------- |
| a@b.com |

**Explanation:** a@b.com is repeated two times.

## Solution

#### 方法一：使用 `GROUP BY` 和临时表

**算法**

重复的电子邮箱存在多次。要计算每封电子邮件的存在次数，我们可以使用以下代码。

```sql
select Email, count(Email) as num
from Person
group by Email;
```

| Email   | num |
| ------- | --- |
| a@b.com | 2   |
| c@d.com | 1   |

以此作为临时表，我们可以得到下面的解决方案。

```sql
select Email from
(
  select Email, count(Email) as num
  from Person
  group by Email
) as statistic
where num > 1
;
```

#### 方法二：使用 `GROUP BY` 和 `HAVING` 条件

向 `GROUP BY` 添加条件的一种更常用的方法是使用 `HAVING` 子句，该子句更为简单高效。

所以我们可以将上面的解决方案重写为：

```sql
select Email
from Person
group by Email
having count(Email) > 1;
```

#### 方法三

```sql
SELECT DISTINCT p1.email AS Email
FROM Person p1,
     Person p2
WHERE p1.id != p2.id
  AND p1.email = p2.email;
```

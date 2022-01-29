---
title: "LeetCode 176. 第二高的薪水"
date: 2022-01-29T10:28:00+08:00
draft: false
tags: ["leetcode", "中等", "数据库"]
math: true
---

```sql
Create table If Not Exists Employee (id int, salary int);
Truncate table Employee;
insert into Employee (id, salary) values ('1', '100');
insert into Employee (id, salary) values ('2', '200');
insert into Employee (id, salary) values ('3', '300');
```

`Employee` 表：

| Column Name | Type |
| ----------- | ---- |
| id          | int  |
| salary      | int  |

id 是这个表的主键。
表的每一行包含员工的工资信息。

编写一个 SQL 查询，获取并返回 `Employee`  表中第二高的薪水 。如果不存在第二高的薪水，查询应该返回 `null` 。

查询结果如下例所示。

<!--more-->

**示例 1：**

**输入：**
Employee 表：

| id  | salary |
| --- | ------ |
| 1   | 100    |
| 2   | 200    |
| 3   | 300    |

**输出：**

| SecondHighestSalary |
| ------------------- |
| 200                 |

**示例 2：**

**输入：**
Employee 表：

| id  | salary |
| --- | ------ |
| 1   | 100    |

**输出：**

| SecondHighestSalary |
| ------------------- |
| null                |

## 题解

#### 方法一：使用子查询和 `LIMIT` 子句

**算法**

将不同的薪资按降序排序，然后使用 [`LIMIT`](https://dev.mysql.com/doc/refman/5.7/en/select.html) 子句获得第二高的薪资。

```sql
SELECT DISTINCT
    Salary AS SecondHighestSalary
FROM
    Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1;
```

然而，如果没有这样的第二最高工资，这个解决方案将被判断为 “错误答案”，因为本表可能只有一项记录。为了克服这个问题，我们可以将其作为临时表。

```sql
SELECT
    (SELECT DISTINCT
            Salary
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT 1 OFFSET 1) AS SecondHighestSalary
;
```

#### 方法二：使用 `IFNULL` 和 `LIMIT` 子句

解决 “NULL” 问题的另一种方法是使用 “IFNULL” 函数，如下所示。

```sql
SELECT
    IFNULL(
      (SELECT DISTINCT Salary
       FROM Employee
       ORDER BY Salary DESC
        LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary;
```

#### 方法三

```sql
SELECT MAX(salary) AS SecondHighestSalary FROM employee WHERE (salary < (SELECT MAX(salary) FROM employee));
```

#### 方法四

```sql
SELECT IFNULL ((SELECT salary FROM employee WHERE (salary < (SELECT salary FROM employee order by salary desc limit 1)) order by salary desc limit 1),NULL) AS SecondHighestSalary;
```

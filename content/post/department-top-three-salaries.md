---
title: "LeetCode 185. Department Top Three Salaries"
date: 2022-02-08T18:41:34+08:00
draft: false
tags: ["leetcode", "困难", "数据库"]
math: true
---

```sql
Create table If Not Exists Employee
(
    id           int,
    name         varchar(255),
    salary       int,
    departmentId int
);
Create table If Not Exists Department
(
    id   int,
    name varchar(255)
);
Truncate table Employee;
insert into Employee (id, name, salary, departmentId)
values ('1', 'Joe', '85000', '1');
insert into Employee (id, name, salary, departmentId)
values ('2', 'Henry', '80000', '2');
insert into Employee (id, name, salary, departmentId)
values ('3', 'Sam', '60000', '2');
insert into Employee (id, name, salary, departmentId)
values ('4', 'Max', '90000', '1');
insert into Employee (id, name, salary, departmentId)
values ('5', 'Janet', '69000', '1');
insert into Employee (id, name, salary, departmentId)
values ('6', 'Randy', '85000', '1');
insert into Employee (id, name, salary, departmentId)
values ('7', 'Will', '70000', '1');
Truncate table Department;
insert into Department (id, name)
values ('1', 'IT');
insert into Department (id, name)
values ('2', 'Sales');
```

Table: `Employee`

| Column Name  | Type    |
| ------------ | ------- |
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |

id is the primary key column for this table.
departmentId is a foreign key of the ID from the `Department` table.
Each row of this table indicates the ID, name, and salary of an employee. It also contains the ID of their department.

Table: `Department`

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| name        | varchar |

id is the primary key column for this table.
Each row of this table indicates the ID of a department and its name.

A company's executives are interested in seeing who earns the most money in each of the company's departments. A **high earner** in a department is an employee who has a salary in the **top three unique** salaries for that department.

Write an SQL query to find the employees who are **high earners** in each of the departments.

Return the result table **in any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Employee table:

| id  | name  | salary | departmentId |
| --- | ----- | ------ | ------------ |
| 1   | Joe   | 85000  | 1            |
| 2   | Henry | 80000  | 2            |
| 3   | Sam   | 60000  | 2            |
| 4   | Max   | 90000  | 1            |
| 5   | Janet | 69000  | 1            |
| 6   | Randy | 85000  | 1            |
| 7   | Will  | 70000  | 1            |

Department table:

| id  | name  |
| --- | ----- |
| 1   | IT    |
| 2   | Sales |

**Output:**

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |

**Explanation:**
In the IT department:

- Max earns the highest unique salary
- Both Randy and Joe earn the second-highest unique salary
- Will earns the third-highest unique salary
  In the Sales department:
- Henry earns the highest salary
- Sam earns the second-highest salary
- There is no third-highest salary as there are only two employees

## 题解

#### 方法一：使用 `JOIN` 和子查询

**算法**

公司里前 3 高的薪水意味着有不超过 3 个工资比这些值大。

```sql
select e1.Name as 'Employee', e1.Salary
from Employee e1
where 3 >
      (
          select count(distinct e2.Salary)
          from Employee e2
          where e2.Salary > e1.Salary
      )
;
```

在这个代码里，我们统计了有多少人的工资比 e1.Salary 高，所以样例的输出应该如下所示。

| Employee | Salary |
| -------- | ------ |
| Henry    | 80000  |
| Max      | 90000  |
| Randy    | 85000  |

然后，我们需要把表 **Employee** 和表 **Department** 连接来获得部门信息。

**MySQL**

```sql
SELECT d.Name  AS 'Department',
       e1.Name AS 'Employee',
       e1.Salary
FROM Employee e1
         JOIN
     Department d ON e1.DepartmentId = d.Id
WHERE 3 > (SELECT COUNT(DISTINCT e2.Salary)
           FROM Employee e2
           WHERE e2.Salary > e1.Salary
             AND e1.DepartmentId = e2.DepartmentId
)
;
```

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Joe      | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |

#### 方法二

```sql
SELECT b.name AS Department, a.name AS Employee, a.salary
FROM (SELECT *, dense_rank() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS r
      FROM Employee) a,
     Department b
WHERE a.r <= 3 && a.departmentId = b.id;
```

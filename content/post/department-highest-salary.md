---
title: "LeetCode 184. Department Highest Salary"
date: 2022-02-07T10:55:28+08:00
draft: false
tags: ["leetcode", "中等", "数据库"]
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
values ('1', 'Joe', '70000', '1');
insert into Employee (id, name, salary, departmentId)
values ('2', 'Jim', '90000', '1');
insert into Employee (id, name, salary, departmentId)
values ('3', 'Henry', '80000', '2');
insert into Employee (id, name, salary, departmentId)
values ('4', 'Sam', '60000', '2');
insert into Employee (id, name, salary, departmentId)
values ('5', 'Max', '90000', '1');
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

Write an SQL query to find employees who have the highest salary in each of the departments.

Return the result table in **any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**

Employee table:

| id  | name  | salary | departmentId |
| --- | ----- | ------ | ------------ |
| 1   | Joe   | 70000  | 1            |
| 2   | Jim   | 90000  | 1            |
| 3   | Henry | 80000  | 2            |
| 4   | Sam   | 60000  | 2            |
| 5   | Max   | 90000  | 1            |

Department table:

| id  | name  |
| --- | ----- |
| 1   | IT    |
| 2   | Sales |

**Output:**

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
| IT         | Max      | 90000  |

**Explanation:** Max and Jim both have the highest salary in the IT department and Henry has the highest salary in the Sales department.

## Solution

因为 **Employee** 表包含 _Salary_ 和 _DepartmentId_ 字段，我们可以以此在部门内查询最高工资。

```sql
SELECT DepartmentId,
       MAX(Salary)
FROM Employee
GROUP BY DepartmentId;
```

> 注意：有可能有多个员工同时拥有最高工资，所以最好在这个查询中不包含雇员名字的信息。

| DepartmentId | MAX(Salary) |
| ------------ | ----------- |
| 1            | 90000       |
| 2            | 80000       |

然后，我们可以把表 **Employee** 和 **Department** 连接，再在这张临时表里用 `IN` 语句查询部门名字和工资的关系。

```sql
SELECT Department.name AS 'Department',
       Employee.name   AS 'Employee',
       Salary
FROM Employee
         JOIN
     Department ON Employee.DepartmentId = Department.Id
WHERE (Employee.DepartmentId, Salary) IN
      (SELECT DepartmentId,
              MAX(Salary)
       FROM Employee
       GROUP BY DepartmentId
      )
;
```

或者

```sql
SELECT c.name   AS Department,
       a.name   AS Employee,
       a.salary AS Salary
FROM Employee a,
     (SELECT departmentID,
             Max(salary) AS salary
      FROM Employee
      GROUP BY departmentID) b,
     Department c
WHERE a.salary = b.salary
  AND a.departmentId = b.departmentId
  AND a.departmentId = c.id;
```

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| Sales      | Henry    | 80000  |
| IT         | Max      | 90000  |

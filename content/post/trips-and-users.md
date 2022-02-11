---
title: "LeetCode 262. Trips and Users"
date: 2022-02-11T11:04:37+08:00
draft: false
tags: ["leetcode", "困难", "数据库"]
math: true
---

Table: `Trips`

| Column Name | Type |
| ----------- | ---- |
| id          | int  |
| client_id   | int  |
| driver_id   | int  |
| city_id     | int  |
| status      | enum |
| request_at  | date |

id is the primary key for this table.
The table holds all taxi trips. Each trip has a unique id, while client_id and driver_id are foreign keys to the users_id at the Users table.
Status is an ENUM type of ('completed', 'cancelled_by_driver', 'cancelled_by_client').

Table: `Users`

| Column Name | Type |
| ----------- | ---- |
| users_id    | int  |
| banned      | enum |
| role        | enum |

users_id is the primary key for this table.
The table holds all users. Each user has a unique users_id, and role is an ENUM type of ('client', 'driver', 'partner').
banned is an ENUM type of ('Yes', 'No').

The **cancellation rate** is computed by dividing the
number of canceled (by client or driver) requests with unbanned users by
the total number of requests with unbanned users on that day.

Write a SQL query to find the **cancellation rate** of requests with unbanned users (**both client and driver must not be banned**) each day between `"2013-10-01"` and `"2013-10-03"`. Round `Cancellation Rate` to **two decimal** points.

Return the result table in **any order**.

<!--more-->

The query result format is in the following example.

**Example 1:**

**Input:**
Trips table:

| id  | client_id | driver_id | city_id | status              | request_at |
| --- | --------- | --------- | ------- | ------------------- | ---------- |
| 1   | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2   | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3   | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4   | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5   | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6   | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7   | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8   | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9   | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10  | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |

Users table:

| users_id | banned | role   |
| -------- | ------ | ------ |
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |

**Output:**

| Day        | Cancellation Rate |
| ---------- | ----------------- |
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |

**Explanation:**
On 2013-10-01:

- There were 4 requests in total, 2 of which were canceled.
- However, the request with Id=2 was made by a banned client (User_Id=2), so it is ignored in the calculation.
- Hence there are 3 unbanned requests in total, 1 of which was canceled.
- The Cancellation Rate is (1 / 3) = 0.33
  On 2013-10-02:
- There were 3 requests in total, 0 of which were canceled.
- The request with Id=6 was made by a banned client, so it is ignored.
- Hence there are 2 unbanned requests in total, 0 of which were canceled.
- The Cancellation Rate is (0 / 2) = 0.00
  On 2013-10-03:
- There were 3 requests in total, 1 of which was canceled.
- The request with Id=8 was made by a banned client, so it is ignored.
- Hence there are 2 unbanned request in total, 1 of which were canceled.
- The Cancellation Rate is (1 / 2) = 0.50

## Solution

统计每天非禁止用户的取消率，需要知道非禁止用户有哪些，总行程数，取消的行程数。

### 解法一

首先确定被禁止用户的行程记录，再剔除这些行程记录。

行程表中，字段 client_id 和 driver_id，都与用户表中的 users_id 关联。因此只要 client_id 和 driver_id 中有一个被禁止了，此条行程记录要被剔除。

先说一种错误的找出没被禁止用户行程记录的方法。此方法很有迷惑性。

思路：

```
if (client_id = users_id 或 driver_id = users_id) 且 users_id没有被禁止
{
    此条记录没被禁止。
}
```

SQL 代码

```sql
SELECT *
FROM Trips AS T
    JOIN Users AS U ON (
        T.client_id = U.users_id
        OR T.driver_id = U.users_id
    )
    AND U.banned = 'No';
```

乍一看，思路是对。其实是错误的。因为，我们不知觉得肯定了一个假设—— client_id 与 driver_id 是相同的。只有当两者相同时，才能用此条件排除被禁止用户的行程记录。

错误的结果：

| Id  | Client_Id | Driver_Id | City_Id | STATUS              | Request_at | Users_Id | Banned | Role   |
| --- | --------- | --------- | ------- | ------------------- | ---------- | -------- | ------ | ------ |
| 1   | 1         | 10        | 1       | completed           | 2013-10-01 | 1        | No     | client |
| 1   | 1         | 10        | 1       | completed           | 2013-10-01 | 10       | No     | driver |
| 2   | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 | 11       | No     | driver |
| 3   | 3         | 12        | 6       | completed           | 2013-10-01 | 3        | No     | client |
| 3   | 3         | 12        | 6       | completed           | 2013-10-01 | 12       | No     | driver |
| 4   | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 | 4        | No     | client |
| 4   | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 | 13       | No     | driver |
| 5   | 1         | 10        | 1       | completed           | 2013-10-02 | 1        | No     | client |
| 5   | 1         | 10        | 1       | completed           | 2013-10-02 | 10       | No     | driver |
| 6   | 2         | 11        | 6       | completed           | 2013-10-02 | 11       | No     | driver |
| 7   | 3         | 12        | 6       | completed           | 2013-10-02 | 3        | No     | client |
| 7   | 3         | 12        | 6       | completed           | 2013-10-02 | 12       | No     | driver |
| 8   | 2         | 12        | 12      | completed           | 2013-10-03 | 12       | No     | driver |
| 9   | 3         | 10        | 12      | completed           | 2013-10-03 | 3        | No     | client |
| 9   | 3         | 10        | 12      | completed           | 2013-10-03 | 10       | No     | driver |
| 10  | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 | 4        | No     | client |
| 10  | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 | 13       | No     | driver |

结果中，被禁止的 users_id = 2，其行程记录没被剔除掉。

明显， client_id 与 driver_id 不一定相同 。

正确的做法是对 client_id 和 driver_id 各自关联的 users_id，同时检测是否被禁止。

```
if (client_id = users_id_1 且 users_id_1没被禁止 并且 client_id = users_id_2 且 users_id_2没被禁止){
    此条记录没被禁止。
}
```

SQL 代码：

```sql
SELECT *
FROM Trips AS T
    JOIN Users AS U1 ON (
        T.client_id = U1.users_id
        AND U1.banned = 'No'
    )
    JOIN Users AS U2 ON (
        T.driver_id = U2.users_id
        AND U2.banned = 'No'
    );
```

在此基础上，按日期分组，统计每组的 总行程数，取消的行程数 。

每组的总行程数：COUNT(T.STATUS)。

每组的取消的行程数：

```sql
SUM(IF(T.STATUS = 'completed', 0, 1)
```

取消率 = 每组的取消的行程数 / 每组的总行程数

完整逻辑为:

```sql
SELECT T.request_at AS `Day`,
    ROUND(
        SUM(IF(T.STATUS = 'completed', 0, 1)) / COUNT(T.STATUS),
        2
    ) AS `Cancellation Rate`
FROM Trips AS T
    JOIN Users AS U1 ON (
        T.client_id = U1.users_id
        AND U1.banned = 'No'
    )
    JOIN Users AS U2 ON (
        T.driver_id = U2.users_id
        AND U2.banned = 'No'
    )
WHERE T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at;
```

其中 SUM 求和函数，COUNT 计数函数，ROUND 四舍五入函数。

### 解法二

思路与解法一相同。而采用不同的方法排除掉被禁止用户的行程记录。想到排除，就联想到集合差。

client_id 和 driver_id 的全部为集合 U。被禁止的 users_id 集合为 A。

U 减去 A 的结果为没被禁止的用户。

```sql
(
    SELECT users_id
    FROM Users
    WHERE banned = 'Yes'
) AS A
```

好了，先演示一个错误的解法：

行程表连接表 A，排除掉被被禁止的行程。

```sql
SELECT *
FROM Trips AS T,
    (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    ) AS A
WHERE (
        T.Client_Id != A.users_id
        AND T.Driver_Id != A.users_id
    );
```

剩下的逻辑与解法一后部分相同，完善后的逻辑为：

```sql
SELECT T.request_at AS `Day`,
    ROUND(
        SUM(IF(T.STATUS = 'completed', 0, 1)) / COUNT(T.STATUS),
        2
    ) AS `Cancellation Rate`
FROM Trips AS T,
    (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    ) AS A
WHERE (
        T.Client_Id != A.users_id
        AND T.Driver_Id != A.users_id
    )
    AND T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at;
```

很可惜，当表 A 为空时，此方法的结果是空表。但是表 A 为空，可能是有用户但是没有被禁止的用户。因此方法是错误的。

正确的解法是：行程表 left join 表 A 两次，A.users_id 都为 NULL 的行都是没被排除的行。

```sql
SELECT *
FROM Trips AS T
    LEFT JOIN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    ) AS A ON (T.Client_Id = A.users_id)
    LEFT JOIN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    ) AS A1 ON (T.Driver_Id = A1.users_id)
WHERE A.users_id IS NULL
    AND A1.users_id IS NULL;
```

补上其它部分的逻辑为：

```sql
SELECT T.request_at AS `Day`,
    ROUND(
        SUM(IF(T.STATUS = 'completed', 0, 1)) / COUNT(T.STATUS),
        2
    ) AS `Cancellation Rate`
FROM Trips AS T
    LEFT JOIN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    ) AS A ON (T.Client_Id = A.users_id)
    LEFT JOIN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    ) AS A1 ON (T.Driver_Id = A1.users_id)
WHERE A.users_id IS NULL
    AND A1.users_id IS NULL
    AND T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at;
```

### 解法三

与解法二思路相同。找出被禁止的用户后，不再连接行程表和用户表，直接从行程表中排除掉被被禁止用户的行程记录。

被禁止的用户用子查询：

```sql
(
    SELECT users_id
    FROM Users
    WHERE banned = 'Yes'
)
```

行程表中 client_id 和 driver_id 都在此子查询结果中的行要剔除掉。

```sql
SELECT *
FROM Trips AS T
WHERE T.Client_Id NOT IN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    )
    AND T.Driver_Id NOT IN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    );
```

补上其它部分：

```sql
SELECT T.request_at AS `Day`,
    ROUND(
        SUM(IF(T.STATUS = 'completed', 0, 1)) / COUNT(T.STATUS),
        2
    ) AS `Cancellation Rate`
FROM Trips AS T
WHERE T.Client_Id NOT IN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    )
    AND T.Driver_Id NOT IN (
        SELECT users_id
        FROM Users
        WHERE banned = 'Yes'
    )
    AND T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at;
```

---
title: "LeetCode 180. 连续出现的数字"
date: 2022-02-03T21:06:00+08:00
draft: false
tags: ["leetcode", "中等", "数据库"]
math: true
---

```sql
Create table If Not Exists Logs (id int, num int);
Truncate table Logs;
insert into Logs (id, num) values ('1', '1');
insert into Logs (id, num) values ('2', '1');
insert into Logs (id, num) values ('3', '1');
insert into Logs (id, num) values ('4', '2');
insert into Logs (id, num) values ('5', '1');
insert into Logs (id, num) values ('6', '2');
insert into Logs (id, num) values ('7', '2');
```

表：`Logs`

| Column Name | Type    |
| ----------- | ------- |
| id          | int     |
| num         | varchar |

id 是这个表的主键。

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

返回的结果表中的数据可以按 **任意顺序** 排列。

<!--more-->

查询结果格式如下面的例子所示：

Logs 表：

| Id  | Num |
| --- | --- |
| 1   | 1   |
| 2   | 1   |
| 3   | 1   |
| 4   | 2   |
| 5   | 1   |
| 6   | 2   |
| 7   | 2   |

Result 表：

| ConsecutiveNums |
| --------------- |
| 1               |

1 是唯一连续出现至少三次的数字。

## 题解

连续出现的意味着相同数字的 Id 是连着的，由于这题问的是至少连续出现 3 次，我们使用 **Logs** 并检查是否有 3 个连续的相同数字。

```sql
SELECT *
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;
```

| Id  | Num | Id  | Num | Id  | Num |
| --- | --- | --- | --- | --- | --- |
| 1   | 1   | 2   | 1   | 3   | 1   |

> 注意：前两列来自 l1 ，接下来两列来自 l2 ，最后两列来自 l3 。

然后我们从上表中选择任意的 _Num_ 获得想要的答案。同时我们需要添加关键字 `DISTINCT` ，因为如果一个数字连续出现超过 3 次，会返回重复元素。

**MySQL**

```sql
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;
```

---
title: "LeetCode 571. Find Median Given Frequency of Numbers"
date: 2022-03-02T11:09:05+08:00
draft: false
tags: ["leetcode", "困难", "数据库"]
math: true
---

Table: `Numbers`

| Column Name | Type |
| ----------- | ---- |
| num         | int  |
| frequency   | int  |

num is the primary key for this table.
Each row of this table shows the frequency of a number in the database.

The [**median**](https://en.wikipedia.org/wiki/Median) is the value separating the higher half from the lower half of a data sample.

Write an SQL query to report the **median** of all the numbers in the database after decompressing the `Numbers` table. Round the median to **one decimal point**.

<!--more-->

The query result format is in the following example.

**Example 1:**

**Input:**
Numbers table:

| num | frequency |
| --- | --------- |
| 0   | 7         |
| 1   | 1         |
| 2   | 3         |
| 3   | 1         |

**Output:**

| median |
| ------ |
| 0.0    |

**Explanation:**
If we decompress the Numbers table, we will get [0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3], so the median is (0 + 0) / 2 = 0.

## Solution

中位数就是将所有数字按照升序或者降序排列，然后取最中间的数字

- 数字个数是奇数的话，那么中位数会在这个序列中
- 数字个数是偶数的话，那么中位数是最中间的两个数的平均值

步骤：

- `sum(frequency) over() total_frequency` 计算出所有数字的个数，这里使用窗口函数 `over()` 就不需要再后面使用 `group by`
  - 计算总数还可以用 `select sum(frequency) as total_frequency from numbers`
- `sum(frequency) over(order by num desc) desc_frequency` 使用窗口函数 `over(order by num desc)` 按照 `num` 降序计算出当前数字和之前数字出现的次数

```sql
SELECT num,
    SUM(frequency) OVER(
        ORDER BY num DESC
    ) desc_frequency
FROM numbers;
```

| num | desc_frequency |
| --- | -------------- |
| 3   | 1              |
| 2   | 4              |
| 1   | 5              |
| 0   | 12             |

- `sum(frequency) over(order by num asc) asc_frequency` 使用窗口函数 `over(order by num asc)` 按照 `num` 升序计算出当前数字和之前数字出现的次数

```sql
SELECT num,
    SUM(frequency) OVER(
        ORDER BY num ASC
    ) asc_frequency
FROM numbers;
```

| num | asc_frequency |
| --- | ------------- |
| 0   | 7             |
| 1   | 8             |
| 2   | 11            |
| 3   | 12            |

- 将查询出来的 `num` ， `desc_frequency` ， `asc_frequency` ， `total_frequency` 作为临时表 `temp`

- 查询临时表 `temp` ， 筛选条件是 `desc_frequency >= total_frequency / 2 and asc_frequency >= total_frequency / 2` ， `desc_frequency` 的一半就是中位数

- 通过筛选条件查询出来的 `num` 就是中位数，使用 `avg` 对其求平均数，因为如果是偶数个的话，查出来的中位数是两个。

```sql
SELECT AVG(num) AS median
FROM (
        SELECT num,
            Frequency,
            sum(Frequency) OVER (
                ORDER BY num DESC
            ) AS desc_frequency,
            sum(Frequency) OVER (
                ORDER BY num
            ) AS asc_frequency,
            sum(Frequency) OVER () AS total_frequency
        FROM Numbers
    ) b
WHERE desc_frequency >= total_frequency / 2
    AND asc_frequency >= total_frequency / 2;
```

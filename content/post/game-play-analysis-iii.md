---
title: "LeetCode 534. Game Play Analysis III"
date: 2022-02-26T09:38:26+08:00
draft: false
tags: ["leetcode", "中等", "数据库"]
math: true
---

| Column Name  | Type |
| ------------ | ---- |
| player_id    | int  |
| device_id    | int  |
| event_date   | date |
| games_played | int  |

(player_id, event_date) is the primary key of this table.
This table shows the activity of players of some games.
Each row is a record of a player who logged in and played a number of games (possibly 0) before logging out on someday using some device.

Write an SQL query to report for each player and date, how many games played **so far** by the player. That is, the total number of games played by the player until that date. Check the example for clarity.

Return the result table in **any order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

**Input:**
Activity table:

| player_id | device_id | event_date | games_played |
| --------- | --------- | ---------- | ------------ |
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 1         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |

**Output:**

| player_id | event_date | games_played_so_far |
| --------- | ---------- | ------------------- |
| 1         | 2016-03-01 | 5                   |
| 1         | 2016-05-02 | 11                  |
| 1         | 2017-06-25 | 12                  |
| 3         | 2016-03-02 | 0                   |
| 3         | 2018-07-03 | 5                   |

**Explanation:**
For the player with id 1, 5 + 6 = 11 games played by 2016-05-02, and 5 + 6 + 1 = 12 games played by 2017-06-25.
For the player with id 3, 0 + 5 = 5 games played by 2018-07-03.
Note that for each player we only care about the days when the player logged in.

## Solution

### SQL：方法一

```sql
select player_id,
    event_date,
    sum(
        case
            when @pre_player_id = player_id then @n := @n + games_played
            when @pre_player_id := player_id then @n := games_played
        end
    ) as games_played_so_far
from (
        select *
        from Activity
        order by player_id,
            event_date
    ) temp,
    (
        select @pre_player_id := null,
            @n := 0
    ) init
group by player_id,
    event_date;
```

#### 解析

因为最终的结果是计算每个用户在某天玩游戏的次数，所以需要按照 `player_id` 和 `event_date` 分组。

因为 `order by` 执行的顺序在 `sum` 函数后面执行，所以这里需要先对 `player_id` 和 `event_date` 先进行排序。

具体实现：

- 将 `activity` 按照 `player_id` 和 `event_date` 升序排序，命名为 `temp` 临时表
- 将临时表 `temp` 按照 `player_id` 和 `event_date` 进行分组
- 使用 `case ... when ... then ...end` 语句对分组后的 `temp` 中 `games_played` 进行输出，并用 `sum` 求和

### SQL：方法二

```sql
select player_id,
    event_date,
    sum(games_played) over (
        partition by player_id
        order by event_date
    ) as games_played_so_far
from Activity;
```

#### 解析

在 `sum` 函数后面可以使用 `over` 对其按照 `player_id` 分组，并按照 `event_date` 排序。

### SQL：方法三

```sql
select a.player_id,
    a.event_date,
    sum(b.games_played) as games_played_so_far
from Activity a
    join Activity b on a.player_id = b.player_id
where a.event_date >= b.event_date
group by a.player_id,
    a.event_date;
```

#### 解析

- 将表 `activity` 自连，连接条件 `a.player_id = b.player_id` 筛选出 `a.event_date >= b.event_date`
- 按照 `a` 表的 `player_id` 和 `a.event_date` 进行分组
- 在求和的时候，使用的是 `b.games_played`
  - 因为筛选条件是 `a.event_date >= b.event_date` 也就是说在 `a.event_date >= b.event_date` 的数据中， `a.games_played` 都是一样的，是不对的。

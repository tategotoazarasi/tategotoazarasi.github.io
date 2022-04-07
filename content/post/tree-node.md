---
title: "LeetCode 608. Tree Node"
date: 2022-04-07T12:05:45+08:00
draft: false
tags: ["leetcode", "简单", "数据库", "组合查询 & 指定选取"]
math: true
---

Table: `Tree`

| Column Name | Type |
| ----------- | ---- |
| id          | int  |
| p_id        | int  |

id is the primary key column for this table.
Each row of this table contains information about the id of a node and the id of its parent node in a tree.
The given structure is always a valid tree.

Each node in the tree can be one of three types:

- **"Leaf"**: if the node is a leaf node.
- **"Root"**: if the node is the root of the tree.
- **"Inner"**: If the node is neither a leaf node nor a root node.

Write an SQL query to report the type of each node in the tree.

Return the result table **ordered** by `id` **in ascending order**.

The query result format is in the following example.

<!--more-->

**Example 1:**

![](/images/608tree1.jpg)

**Input:**
Tree table:

| id  | p_id |
| --- | ---- |
| 1   | null |
| 2   | 1    |
| 3   | 1    |
| 4   | 2    |
| 5   | 2    |

**Output:**

| id  | type  |
| --- | ----- |
| 1   | Root  |
| 2   | Inner |
| 3   | Leaf  |
| 4   | Leaf  |
| 5   | Leaf  |

**Explanation:**
Node 1 is the root node because its parent node is null and it has child nodes 2 and 3.
Node 2 is an inner node because it has parent node 1 and child node 4 and 5.
Nodes 3, 4, and 5 are leaf nodes because they have parent nodes and they do not have child nodes.

**Example 2:**

![](/images/608tree2.jpg)

**Input:**
Tree table:

| id  | p_id |
| --- | ---- |
| 1   | null |

**Output:**

| id  | type |
| --- | ---- |
| 1   | Root |

**Explanation:** If there is only one node on the tree, you only need to output its root attributes.

#### 方法 1：使用 `UNION`

**想法**我们可以按照下面的定义，求出每一条记录的节点类型。

- Root: 没有父节点
- Inner: 它是某些节点的父节点，且有非空的父节点
- Leaf: 除了上述两种情况以外的节点

**算法**将上述定义转化，我们可以得到下面的代码。

根节点是没有父节点的节点。

```sql
SELECT id,
       'Root' AS Type
FROM tree
WHERE p_id IS NULL;
```

叶子节点是没有孩子节点的节点，且它有父亲节点。

```sql
SELECT id,
       'Leaf' AS Type
FROM tree
WHERE id NOT IN (
    SELECT DISTINCT p_id
    FROM tree
    WHERE p_id IS NOT NULL
)
  AND p_id IS NOT NULL;
```

内部节点是有孩子节点和父节点的节点。

```sql
SELECT id,
       'Inner' AS Type
FROM tree
WHERE id IN (
    SELECT DISTINCT p_id
    FROM tree
    WHERE p_id IS NOT NULL
)
  AND p_id IS NOT NULL;
```

所以本题的一种解法是将这些情况用 `UNION` 合并起来。

```sql
SELECT id,
       'Root' AS Type
FROM tree
WHERE p_id IS NULL
UNION
SELECT id,
       'Leaf' AS Type
FROM tree
WHERE id NOT IN (
    SELECT DISTINCT p_id
    FROM tree
    WHERE p_id IS NOT NULL
)
  AND p_id IS NOT NULL
UNION
SELECT id,
       'Inner' AS Type
FROM tree
WHERE id IN (
    SELECT DISTINCT p_id
    FROM tree
    WHERE p_id IS NOT NULL
)
  AND p_id IS NOT NULL
ORDER BY id;
```

#### 方法 II：使用流控制语句 `CASE`

**算法**

与上面解法类似，本解法使用流控制语句，流控制语句对基于不同输入产生不同输出非常有效。

本方法中，我们使用 [`CASE`](https://dev.mysql.com/doc/refman/5.7/en/case.html) 语句。

```sql
SELECT id      AS `Id`,
       CASE
           WHEN tree.id = (
               SELECT atree.id
               FROM tree atree
               WHERE atree.p_id IS NULL
           ) THEN 'Root'
           WHEN tree.id IN (
               SELECT atree.p_id
               FROM tree atree
           ) THEN 'Inner'
           ELSE 'Leaf'
           END AS Type
FROM tree
ORDER BY `Id`;
```

> MySQL 除了 `CASE` 语句以外还提供了不同的流控制语句。

你可以尝试将上面的方法用 [`IF`](https://dev.mysql.com/doc/refman/5.7/en/control-flow-functions.html#function_if) 重写。

#### 方法 III；使用 `IF` 函数

**算法**

我们还可以使用 [`IF`](https://dev.mysql.com/doc/refman/5.7/en/control-flow-functions.html#function_if) 函数来避免复杂的流控制语句。

```sql
SELECT atree.id,
       IF(
               ISNULL(atree.p_id),
               'Root',
               IF(
                           atree.id IN (
                           SELECT p_id
                           FROM tree
                       ),
                           'Inner',
                           'Leaf'
                   )
           ) Type
FROM tree atree
ORDER BY atree.id;
```

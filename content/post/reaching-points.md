---
title: "LeetCode 780. Reaching Points"
date: 2022-04-09T14:26:35+08:00
draft: false
tags: ["leetcode", "困难", "数学"]
math: true
---

Given four integers `sx`, `sy`, `tx`, and `ty`, return `true` _if it is possible to convert the point_ `(sx, sy)` _to the point_ `(tx, ty)` _through some operations\*\*, or_ `false` _otherwise_.

The allowed operation on some point `(x, y)` is to convert it to either `(x, x + y)` or `(x + y, y)`.

<!--more-->

**Example 1:**

> **Input:** sx = 1, sy = 1, tx = 3, ty = 5
>
> **Output:** true
>
> **Explanation:**
> One series of moves that transforms the starting point to the target is:
> (1, 1) -> (1, 2)
> (1, 2) -> (3, 2)
> (3, 2) -> (3, 5)

**Example 2:**

> **Input:** sx = 1, sy = 1, tx = 2, ty = 2
>
> **Output:** false

**Example 3:**

> **Input:** sx = 1, sy = 1, tx = 1, ty = 1
>
> **Output:** true

**Constraints:**

- \\(1 <= sx, sy, tx, ty <= 10^9\\)

## Solution

如果从 \\((\textit{sx}, \textit{sy})\\) 开始正向计算，则可能的情况非常多，会超出时间限制。

注意到 \\(\textit{sx}, \textit{sy}, \textit{tx}, \textit{ty}\\) 都是正整数，因此对于给定的状态 \\((\textit{tx}, \textit{ty})\\)，只有当 \\(\textit{tx} \ne \textit{ty}\\) 时才存在上一个状态，且上一个状态唯一，可能的情况如下：

- 如果 \\(\textit{tx} = \textit{ty}\\)，不存在上一个状态，状态 \\((\textit{tx}, \textit{ty})\\) 即为起点状态

- 如果 \\(\textit{tx} > \textit{ty}\\)，则上一个状态是 \\((\textit{tx} - \textit{ty}, \textit{ty})\\)

- 如果 \\(\textit{tx} < \textit{ty}\\)，则上一个状态是 \\((\textit{tx}, \textit{ty} - \textit{tx})\\)。

因此可以从 \\((\textit{tx}, \textit{ty})\\) 开始反向计算，判断是否可以到达状态 \\((\textit{sx}, \textit{sy})\\)。

当 \\(\textit{tx} > \textit{sx}, \textit{ty} > \textit{sy}, \textit{tx} \ne \textit{ty}\\) 三个条件同时成立时，执行反向操作，每一步操作更新 \\((\textit{tx}, \textit{ty})\\) 的值，直到反向操作的条件不成立。

由于每一步反向操作一定是将 \\(\textit{tx}\\) 和 \\(\textit{ty}\\) 中的较大的值减小，因此当 \\(\textit{tx} > \textit{ty}\\) 时可以直接将 \\(\textit{tx}\\) 的值更新为 \\(\textit{tx} \bmod \textit{ty}\\)，当 \\(\textit{tx} < \textit{ty}\\) 时可以直接将 \\(\textit{ty}\\) 的值更新为 \\(\textit{ty} \bmod \textit{tx}\\)。

当反向操作的条件不成立时，根据 \\(\textit{tx}\\) 和 \\(\textit{ty}\\) 的不同情况分别判断是否可以从起点转换到终点。

- 如果 \\(\textit{tx} = \textit{sx}\\) 且 \\(\textit{ty} = \textit{sy}\\)，则已经到达起点状态，因此可以从起点转换到终点。

- 如果 \\(\textit{tx} = \textit{sx}\\) 且 \\(\textit{ty} \ne \textit{sy}\\)，则 \\(\textit{tx}\\) 不能继续减小，只能减小 \\(\textit{ty}\\)，因此只有当 \\(\textit{ty} > \textit{sy}\\) 且 \\((\textit{ty} - \textit{sy}) \bmod \textit{tx} = 0\\) 时可以从起点转换到终点。

- 如果 \\(\textit{ty} = \textit{sy}\\) 且 \\(\textit{tx} \ne \textit{sx}\\)，则 \\(\textit{ty}\\) 不能继续减小，只能减小 \\(\textit{tx}\\)，因此只有当 \\(\textit{tx} > \textit{sx}\\) 且 \\((\textit{tx} - \textit{sx}) \bmod \textit{ty} = 0\\) 时可以从起点转换到终点。

- 如果 \\(\textit{tx} \ne \textit{sx}\\) 且 \\(\textit{ty} \ne \textit{sy}\\)，则不可以从起点转换到终点。

**复杂度分析**

- 时间复杂度：\\(O(\log \max(\textit{tx}, \textit{ty}))\\)，其中 \\(\textit{tx}\\) 和 \\(\textit{ty}\\) 是终点值。

反向计算的过程与辗转相除法相似，时间复杂度是 \\(O(\log \max(\textit{tx}, \textit{ty}))\\)。

- 空间复杂度：\\(O(1)\\)。

```cpp
class Solution {
public:
    bool reachingPoints(int sx, int sy, int tx, int ty) {
        while(tx > sx && ty > sy && tx != ty) {
            if(tx > ty) {
                tx %= ty;
            } else {
                ty %= tx;
            }
        }
        if(tx == sx && ty == sy) {
            return true;
        }
        if(tx == sx) {
            return ty > sy && (ty - sy) % tx == 0;
        }
        if(ty == sy) {
            return tx > sx && (tx - sx) % ty == 0;
        }
        return false;
    }
};
```

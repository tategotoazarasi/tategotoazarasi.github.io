---
title: climbing-stairs
tags: [Import-7704]
created: '2021-12-18T11:18:20.411Z'
modified: '2021-12-30T13:58:00.262Z'
---

---
title: "LeetCode 70. 爬楼梯"
date: 2021-12-12T16:51:18+08:00
draft: false
tags: ["leetcode", "简单", "记忆化搜索", "数学", "动态规划"]
math: true
---

假设你正在爬楼梯。需要 *n*  阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

<!--more-->

**注意：** 给定 _n_ 是一个正整数。

**示例 1：**

> **输入：** 2
>
> **输出：** 2
>
> **解释：** 有两种方法可以爬到楼顶。

1. 1 阶 + 1 阶
2. 2 阶

**示例 2：**

> **输入：** 3
>
> **输出：** 3
>
> **解释：** 有三种方法可以爬到楼顶。

1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶

**提示：**
为了达到第 n 步，你之前的步骤可能是什么？(考虑一下步骤的大小)

```java
class Solution {
    public int climbStairs(int n) {
        if (n == 1) {
            return 1;
        }
        if (n == 2) {
            return 2;
        }
        if (n == 3) {
            return 3;
        }
        int a = 2;
        int b = 3;
        for (int i = 4; i <= n; i++) {
            int temp = a + b;
            a = b;
            b = temp;
        }
        return b;
    }
}
```

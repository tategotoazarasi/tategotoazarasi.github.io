---
title: "LeetCode 7. 整数反转"
date: 2021-12-03T08:29:41+08:00
draft: false
tags: ["leetcode", "简单", "数学"]
math: true
---

给你一个 32 位的有符号整数 `x` ，返回将 `x` 中的数字部分反转后的结果。

如果反转后整数超过 32 位的有符号整数的范围  [\\(-2^{31}\\), \\(2^{31}-1\\)] ，就返回 0。

<!--more-->

**假设环境不允许存储 64 位整数（有符号或无符号）。**

**示例 1：**

> **输入：** x = 123
>
> **输出：** 321

**示例 2：**

> **输入：** x = -123
>
> **输出：** -321

**示例 3：**

> **输入：** x = 120
>
> **输出：** 21

**示例 4：**

> **输入：** x = 0
>
> **输出：** 0

**提示：**

- \\(-2^{31} <= x <= 2^{31}-1\\)

```java
class Solution {
    public int reverse(int x) {
        String s = Integer.toString(Math.abs(x));
        StringBuilder sb = new StringBuilder();
        for (int i = s.length() - 1; i >= 0; i--) {
            sb.append(s.charAt(i));
        }
        try {
            int res = Integer.parseInt(sb.toString());
            return x > 0 ? res : -res;
        } catch (NumberFormatException e) {
            return 0;
        }
    }
}
```

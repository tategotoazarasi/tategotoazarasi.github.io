---
title: "LeetCode 9. 回文数"
date: 2021-12-04T10:03:00+08:00
draft: false
tags: ["leetcode", "简单", "数学"]
math: true
---

给你一个整数 `x` ，如果 `x` 是一个回文整数，返回 `true` ；否则，返回 `false` 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。例如，`121` 是回文，而 `123` 不是。

<!--more-->

**示例 1：**

> **输入：** x = 121
> 
> **输出：** true

**示例 2：**

> **输入：** x = -121
> 
> **输出：** false
> 
> **解释：** 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。

**示例 3：**

> **输入：** x = 10
> 
> **输出：** false
> 
> **解释：** 从右向左读, 为 01 。因此它不是一个回文数。

**示例 4：**

> **输入：** x = -101
> 
> **输出：** false

**提示：**

- \\(-2^{31} <= x <= 2^{31}-1\\)

**进阶：** 你能不将整数转为字符串来解决这个问题吗？

**隐藏提示：** 当你反转整数时，要小心溢出。

```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0) {
            return false;
        }
        if (x == 0) {
            return true;
        }
        int lenx = 1;
        int x2 = x / 10;
        while (x2 != 0) {
            lenx++;
            x2 /= 10;
        }
        x2 = x;
        for (int len = lenx; len > 1; len -= 2) {
            int l = (x2 / (int) (Math.pow(10, len - 1)));
            int r = x2 % 10;
            if (l != r) {
                return false;
            }
            x2 = x2 - l * (int) (Math.pow(10, len - 1));
            x2 = (x2 - r) / 10;
        }
        return true;
    }
}
```
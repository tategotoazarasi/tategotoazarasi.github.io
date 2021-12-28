---
title: "LeetCode 5963. 反转两次的数字"
date: 2021-12-26T15:10:47+08:00
draft: false
tags: ["leetcode", "简单"]
math: true
---

**反转** 一个整数意味着倒置它的所有位。

- 例如，反转 `2021` 得到 `1202` 。反转 `12300` 得到 `321` ，**不保留前导零** 。

给你一个整数 `num` ，**反转** `num` 得到 `reversed1` ，**接着反转** `reversed1` 得到 `reversed2` 。如果 `reversed2` 等于 `num` ，返回 `true` ；否则，返回 `false` 。

<!--more-->

**示例 1：**

**输入：** num = 526
**输出：** true
**解释：** 反转 num 得到 625 ，接着反转 625 得到 526 ，等于 num 。

**示例 2：**

**输入：** num = 1800
**输出：** false
**解释：** 反转 num 得到 81 ，接着反转 81 得到 18 ，不等于 num 。

**示例 3：**

**输入：** num = 0
**输出：** true
**解释：** 反转 num 得到 0 ，接着反转 0 得到 0 ，等于 num 。

**提示：**

- \\(0 <= num <= 106\\)
- 除了数字0本身之外，任何以0结尾的数字在反转时都会永久失去一些数字。

```java
class Solution {
    public boolean isSameAfterReversals(int num) {
        if (num == 0) {
            return true;
        }
        return num % 10 != 0;
    }
}
```

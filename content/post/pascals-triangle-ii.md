---
title: "LeetCode 119. 杨辉三角 II"
date: 2021-12-20T11:13:00+08:00
draft: false
tags: ["leetcode", "简单", "数组", "动态规划"]
math: true
---

给定一个非负索引 `rowIndex`，返回「杨辉三角」的第 `rowIndex` 行。

在「杨辉三角」中，每个数是它左上方和右上方的数的和。

<!--more-->

![](https://tategotoazarasi.github.io/images/1626927345-DZmfxB-PascalTriangleAnimated2.gif)

**示例 1:**

> **输入:** rowIndex = 3
> 
> **输出:** [1,3,3,1]

**示例 2:**

> **输入:** rowIndex = 0
> 
> **输出:** [1]

**示例 3:**

> **输入:** rowIndex = 1
> 
> **输出:** [1,1]

**提示:**

- `0 <= rowIndex <= 33`

**进阶：**

你可以优化你的算法到 `O(rowIndex)` 空间复杂度吗？

```java
class Solution {
    public List<Integer> getRow(int rowIndex) {
        List<Integer> ret = new ArrayList<>(rowIndex + 1);
        ret.add(1);
        for (int i = 1; i <= rowIndex ; i++) {
            ret.add((int) ((long) ret.get(i - 1) * (rowIndex - i + 1) / i));
        }
        return ret;
    }
}
```

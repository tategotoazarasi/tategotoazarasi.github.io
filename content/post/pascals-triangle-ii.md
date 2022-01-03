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

## 题解

由组合数公式 \\( C_n^m = \frac{n!}{m!(n-m)!} \\) ，可以得到同一行的相邻组合数的关系

\\(C_n^m = C_n^{m−1} \times \frac{n−m+1}{m}\\)

由于 \\( C_n^0 = 1 \\)，利用上述公式我们可以在线性时间计算出第 n 行的所有组合数。

**复杂度分析**

- 时间复杂度：O(rowIndex)。

- 空间复杂度：O(1)。不考虑返回值的空间占用。

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

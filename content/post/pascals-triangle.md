---
title: "LeetCode 118. 杨辉三角"
date: 2021-12-19T13:19:32+08:00
draft: false
tags: ["leetcode", "简单", "数组", "动态规划"]
math: true
---

给定一个非负整数  *`numRows`，* 生成「杨辉三角」的前  *`numRows`* 行。

在「杨辉三角」中，每个数是它左上方和右上方的数的和。

<!--more-->

![](https://tategotoazarasi.github.io/images/1626927345-DZmfxB-PascalTriangleAnimated2.gif)

**示例 1:**

> **输入:** numRows = 5
> **输出:** [[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]

**示例  2:**

> **输入:** numRows = 1
> **输出:** [[1]]

**提示:**

- `1 <= numRows <= 30`

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> ret = new ArrayList<>(numRows);
        for (int i = 0; i < numRows; i++) {
            List<Integer> row = new ArrayList<>(i + 1);
            for (int j = 0; j < i + 1; j++) {
                if (j == 0 || j == i) {
                    row.add(1);
                } else {
                    row.add(ret.get(i - 1).get(j - 1) + ret.get(i - 1).get(j));
                }
            }
            ret.add(i, row);
        }
        return ret;
    }
}
```

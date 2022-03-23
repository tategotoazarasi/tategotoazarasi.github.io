---
title: "LeetCode 6023. Minimum White Tiles After Covering With Carpets"
date: 2022-03-20T17:49:37+08:00
draft: false
tags: ["leetcode", "困难", "动态规划"]
math: true
---

You are given a **0-indexed binary** string `floor`, which represents the colors of tiles on a floor:

- `floor[i] = '0'` denotes that the `ith` tile of the floor is colored **black**.
- On the other hand, `floor[i] = '1'` denotes that the `ith` tile of the floor is colored **white**.

You are also given `numCarpets` and `carpetLen`. You have `numCarpets` **black** carpets, each of length `carpetLen` tiles. Cover the tiles with the given carpets such that the number of **white** tiles still visible is **minimum**. Carpets may overlap one another.

Return *the **minimum** number of white tiles still visible.*

<!--more-->

**Example 1:**

![](/images/6023ex1-1.png)

> **Input:** floor = "10110101", numCarpets = 2, carpetLen = 2
>
> **Output:** 2
>
> **Explanation:**
> The figure above shows one way of covering the tiles with the carpets such that only 2 white tiles are visible.
> No other way of covering the tiles with the carpets can leave less than 2 white tiles visible.

**Example 2:**

![](/images/6023ex2.png)

> **Input:** floor = "11111", numCarpets = 2, carpetLen = 3
>
> **Output:** 0
>
> **Explanation:**
> The figure above shows one way of covering the tiles with the carpets such that no white tiles are visible.
> Note that the carpets are able to overlap one another.

**Constraints:**

- `1 <= carpetLen <= floor.length <= 1000`
- `floor[i]` is either `'0'` or `'1'`.
- `1 <= numCarpets <= 1000`
- Can you think of a DP solution?
- Let DP[i][j] denote the minimum number of white tiles still visible from indices i to floor.length-1 after covering with at most j carpets.
- The transition will be whether to put down the carpet at position i (if possible), or not.

## Solution

定义 f[i][j] 表示用 i 条地毯覆盖前 j 块板砖时，没被覆盖的白色砖块的最少数目。

转移时可以考虑是否用第 i 条地毯的末尾覆盖第 j 块板砖：

- 不覆盖：f[i][j]=f[i][j−1]+[floor[j]=‘1’]；
- 覆盖：f[i][j]=f[i−1][j−carpetLen]

取二者最小值。

注意 i=0 的时候只能不覆盖，需要单独计算。

最后答案为 f[numCarpets][floor.length−1]。

```cpp
class Solution {
public:
    int minimumWhiteTiles(string floor, int numCarpets, int carpetLen) {
        int n = floor.length();
        vector<vector<int>> f(numCarpets + 1, vector<int>(n));
        f[0][0] = floor[0] % 2;
        for(int i = 1; i < n; ++i) {
            f[0][i] = f[0][i - 1] + floor[i] % 2;
        }
        for(int i = 1; i <= numCarpets; ++i) {
            // j < carpetLen 的 f[i][j] 均为 0
            for(int j = carpetLen; j < n; ++j) {
                f[i][j] = min(f[i][j - 1] + floor[j] % 2, f[i - 1][j - carpetLen]);
            }
        }
        return f[numCarpets][n - 1];
    }
};
```

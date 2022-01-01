---
title: "LeetCode 2022. 将一维数组转变成二维数组"
date: 2022-01-01T08:24:37+08:00
draft: false
tags: ["leetcode", "简单", "数组", "矩阵", "模拟"]
math: true
---

给你一个下标从 **0**  开始的一维整数数组  `original`  和两个整数  `m`  和   `n` 。你需要使用  `original`  中  **所有**  元素创建一个  `m`  行  `n`  列的二维数组。

`original`  中下标从 `0`  到 `n - 1` （都 **包含** ）的元素构成二维数组的第一行，下标从 `n`  到 `2 * n - 1` （都 **包含** ）的元素构成二维数组的第二行，依此类推。

请你根据上述过程返回一个 `m x n`  的二维数组。如果无法构成这样的二维数组，请你返回一个空的二维数组。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/image-20210826114243-1.png)

> **输入：** original = [1,2,3,4], m = 2, n = 2
>
> **输出：** [[1,2],[3,4]]
>
> **解释：** 构造出的二维数组应该包含 2 行 2 列。
> original 中第一个 n=2 的部分为 [1,2] ，构成二维数组的第一行。
> original 中第二个 n=2 的部分为 [3,4] ，构成二维数组的第二行。

**示例 2：**

> **输入：** original = [1,2,3], m = 1, n = 3
>
> **输出：** [[1,2,3]]
>
> **解释：**
> 构造出的二维数组应该包含 1 行 3 列。
> 将 original 中所有三个元素放入第一行中，构成要求的二维数组。

**示例 3：**

> **输入：** original = [1,2], m = 1, n = 1
>
> **输出：** []
>
> **解释：** original 中有 2 个元素。
> 无法将 2 个元素放入到一个 1x1 的二维数组中，所以返回一个空的二维数组。

**示例 4：**

> **输入：** original = [3], m = 1, n = 2
>
> **输出：** []
>
> **解释：**
> original 中只有 1 个元素。
> 无法将 1 个元素放满一个 1x2 的二维数组，所以返回一个空的二维数组。

**提示：**

- \\(1 <= original.length <= 5 \* 10^4\\)
- \\(1 <= original[i] <= 10^5\\)
- \\(1 <= m, n <= 4 \* 10^4\\)
- 什么时候可以将原件转化为二维数组，什么时候不可能？
- 当且仅当 m \* n == original.length 时，才有可能。
- 如果有可能将原始数据转换为二维数组，请保留一个索引 i，以便将原始数据[i]作为下一个元素添加到二维数组中。

```cpp
class Solution {
    public:
        vector < vector < int >> construct2DArray(vector < int > & original, int m, int n) {
            if (original.size() != m * n) {
                return {};
            }
            auto ans = vector < vector < int >> ();
            int count = 0;
            for (int i = 0; i < m; i++) {
                auto row = vector < int > ();
                for (int j = 0; j < n; j++) {
                    row.push_back(original[count++]);
                }
                ans.push_back(row);
            }
            return ans;
        }
};
```

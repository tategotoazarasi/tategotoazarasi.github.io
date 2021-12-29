---
title: "LeetCode 1995. 统计特殊四元组"
date: 2021-12-29T10:54:22+08:00
draft: false
tags: ["leetcode", "简单", "数学", "字符串"]
math: true
---

给你一个 **下标从 0 开始** 的整数数组 `nums` ，返回满足下述条件的 **不同** 四元组 `(a, b, c, d)` 的 **数目** ：

- `nums[a] + nums[b] + nums[c] == nums[d]` ，且
- `a < b < c < d`

<!--more-->

**示例 1：**

> **输入：** nums = [1,2,3,6]
> 
> **输出：** 1
> 
> **解释：** 满足要求的唯一一个四元组是 (0, 1, 2, 3) 因为 1 + 2 + 3 == 6 。

**示例 2：**

> **输入：** nums = [3,3,6,4,5]
> 
> **输出：** 0
> 
> **解释：** [3,3,6,4,5] 中不存在满足要求的四元组。

**示例 3：**

> **输入：** nums = [1,1,1,3,5]
> 
> **输出：** 4
> 
> **解释：** 满足要求的 4 个四元组如下：

- (0, 1, 2, 3): 1 + 1 + 1 == 3
- (0, 1, 3, 4): 1 + 1 + 3 == 5
- (0, 2, 3, 4): 1 + 1 + 3 == 5
- (1, 2, 3, 4): 1 + 1 + 3 == 5

**提示：**

- `4 <= nums.length <= 50`
- `1 <= nums[i] <= 100`
- N 是非常小的，我们怎么能用呢？
- 我们可以检查每一个可能的四元组吗？

```cpp
class Solution {
    public:
        int countQuadruplets(vector < int > & nums) {
            int sum = 0;
            for (int i = 3; i < nums.size(); i++) {
                for (int j = 0; j < i - 2; j++) {
                    if (j == i) {
                        continue;
                    }
                    for (int k = j + 1; k < i - 1; k++) {
                        if (k == i) {
                            continue;
                        }
                        for (int l = k + 1; l < i; l++) {
                            if (l == i) {
                                continue;
                            }
                            if (nums[k] + nums[j] + nums[l] == nums[i]) {
                                sum++;
                            }
                        }
                    }
                }
            }
            return sum;
        }
};
```

---
title: maximum-subarray
tags: [Import-7704]
created: '2021-12-18T11:18:20.411Z'
modified: '2021-12-30T13:58:02.681Z'
---

---
title: "LeetCode 53. 最大子数组和"
date: 2021-12-08T10:12:55+08:00
draft: false
tags: ["leetcode", "简单", "数组", "分治", "动态规划"]
math: true
---

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

<!--more-->

**子数组** 是数组中的一个连续部分。

**示例 1：**

> **输入：** nums = [-2,1,-3,4,-1,2,1,-5,4]
> 
> **输出：** 6
> 
> **解释：** 连续子数组  [4,-1,2,1] 的和最大，为  6 。

**示例 2：**

> **输入：** nums = [1]
> 
> **输出：** 1

**示例 3：**

> **输入：** nums = [5,4,-1,7,8]
> 
> **输出：** 23

**提示：**

- \\(1 <= nums.length <= 10^5\\)
- \\(-10^4 <= nums[i] <= 10^4\\)

**进阶：** 如果你已经实现复杂度为 `O(n)` 的解法，尝试使用更为精妙的 **分治法** 求解。

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int start = -1;
        int max = nums[0];
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0) {
                start = i;
                break;
            }
            max = Math.max(max, nums[i]);
        }
        if (start == -1) {
            return max;
        }

        max = nums[start];
        int cur = 0;
        for (int i = start; i < nums.length; i++) {
            if (cur < 0) {
                cur = nums[i];
            } else {
                cur += nums[i];
            }
            max = Math.max(cur, max);
        }
        return max;
    }
}
```

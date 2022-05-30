---
title: "LeetCode 713. Subarray Product Less Than K"
date: 2022-05-30T13:24:11+08:00
draft: false
tags: ["leetcode", "中等", "数组", "滑动窗口"]
math: true
---

Given an array of integers `nums` and an integer `k`, return _the number of contiguous subarrays where the product of all the elements in the subarray is strictly less than_ `k`.

<!--more-->

**Example 1:**

    Input: nums = [10,5,2,6], k = 100
    Output: 8
    Explanation: The 8 subarrays that have product less than 100 are:
    [10], [5], [2], [6], [10, 5], [5, 2], [2, 6], [5, 2, 6]
    Note that [10, 5, 2] is not included as the product of 100 is not strictly less than k.

**Example 2:**

    Input: nums = [1,2,3], k = 0
    Output: 0

**Constraints:**

- \\(1 <= nums.length <= 3 \* 10^4\\)
- `1 <= nums[i] <= 1000`
- \\(0 <= k <= 10^6\\)

## 题解

我们固定子数组 \\([i, j]\\) 的右端点 \\(j\\) 时，显然左端点 \\(i\\) 越大，子数组元素乘积越小。

对于子数组 \\([i, j]\\)，当左端点 \\(i \ge l_1\\) 时，所有子数组的元素乘积都小于 \\(k\\)，当左端点 \\(i \lt l_1\\) 时，所有子数组的元素乘积都大于等于 \\(k\\)。

那么对于右端点为 \\(j + 1\\) 的所有子数组，它的左端点 \\(i\\) 就不需要从 \\(0\\) 开始枚举，因为对于所有 \\(i \lt l_1\\) 的子数组，它们的元素乘积都大于等于 \\(k\\)。

我们只要从 \\(i = l_1\\) 处开始枚举，直到子数组 \\(i = l_2\\) 时子数组 \\([l_2, j + 1]\\) 的元素乘积小于 \\(k\\)，那么左端点 \\(i \ge l_2\\) 所有子数组的元素乘积都小于 \\(k\\)。

根据上面的分析，我们枚举子数组的右端点 \\(j\\)，并且左端点从 \\(i = 0\\) 开始，用 \\(\textit{prod}\\) 记录子数组 \\([i, j]\\) 的元素乘积。

每枚举一个右端点 \\(j\\)，如果当前子数组元素乘积 \\(\textit{prod}\\) 大于等于 \\(k\\)，那么我们右移左端点 \\(i\\) 直到满足当前子数组元素乘积小于 \\(k\\) 或者 \\(i > j\\)，那么元素乘积小于 \\(k\\) 的子数组数目为 \\(j - i + 1\\)。

返回所有数目之和。

> \\(\textit{prod}\\) 的值始终不超过 \\(k \times \max_l \{\textit{nums}[l] \}\\)，因此无需担心整型溢出的问题。

**复杂度分析**

- 时间复杂度：\\(O(n)\\)，其中 \\(n\\) 是数组 \\(\textit{nums}\\) 的长度。

两个端点 \\(i\\) 和 \\(j\\) 的增加次数都不超过 \\(n\\)。

- 空间复杂度：\\(O(1)\\)。

```cpp
class Solution {
public:
    int numSubarrayProductLessThanK(vector<int>& nums, int k) {
        int ans  = 0;
        int prod = 1, i = 0;
        for(int j = 0; j < nums.size(); j++) {
            prod *= nums[j];
            while(i <= j && prod >= k) {
                prod /= nums[i];
                i++;
            }
            ans += j - i + 1;
        }
        return ans;
    }
};
```

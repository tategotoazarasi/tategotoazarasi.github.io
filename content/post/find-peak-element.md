---
title: "LeetCode 162. Find Peak Element"
date: 2022-05-27T12:47:10+08:00
draft: false
tags: ["leetcode", "中等", "数组", "二分查找"]
math: true
---

A peak element is an element that is strictly greater than its neighbors.

Given an integer array `nums`, find a peak element, and return its index. If the array contains multiple peaks, return the index to **any of the peaks**.

You may imagine that `nums[-1] = nums[n] = -∞`.

You must write an algorithm that runs in `O(log n)` time.

<!--more-->

**Example 1:**

    Input: nums = [1,2,3,1]
    Output: 2
    Explanation: 3 is a peak element and your function should return the index number 2.

**Example 2:**

    Input: nums = [1,2,1,3,5,6,4]
    Output: 5
    Explanation: Your function can return either index number 1 where the peak element is 2, or index number 5 where the peak element is 6.

**Constraints:**

- `1 <= nums.length <= 1000`
- \\(-2^{31} <= nums[i] <= 2^{31} - 1\\)
- `nums[i] != nums[i + 1]` for all valid `i`.

## Solution

俗话说「人往高处走，水往低处流」。

如果我们从一个位置开始，不断地向高处走，那么最终一定可以到达一个峰值位置。

因此，我们首先在\\([0, n)\\)的范围内随机一个初始位置\\(i\\)，随后根据\\(\textit{nums}[i-1], \textit{nums}[i], \textit{nums}[i+1]\\)三者的关系决定向哪个方向走：

- 如果\\(\textit{nums}[i-1] < \textit{nums}[i] > \textit{nums}[i+1]\\)，那么位置\\(i\\)就是峰值位置，我们可以直接返回\\(i\\)作为答案；

- 如果\\(\textit{nums}[i-1] < \textit{nums}[i] < \textit{nums}[i+1]\\)，那么位置\\(i\\)处于上坡，我们需要往右走，即\\(i \leftarrow i+1\\)；

- 如果\\(\textit{nums}[i-1] > \textit{nums}[i] > \textit{nums}[i+1]\\)，那么位置\\(i\\)处于下坡，我们需要往左走，即\\(i \leftarrow i-1\\)；

- 如果\\(\textit{nums}[i-1] > \textit{nums}[i] < \textit{nums}[i+1]\\)，那么位置\\(i\\)位于山谷，两侧都是上坡，我们可以朝任意方向走。

如果我们规定对于最后一种情况往右走，那么当位置\\(i\\)不是峰值位置时：

- 如果\\(\textit{nums}[i] < \textit{nums}[i+1]\\)，那么我们往右走；

- 如果\\(\textit{nums}[i] > \textit{nums}[i+1]\\)，那么我们往左走。

我们可以发现，如果\\(\textit{nums}[i] < \textit{nums}[i+1]\\)，并且我们从位置\\(i\\)向右走到了位置\\(i+1\\)，那么位置\\(i\\)左侧的所有位置是不可能在后续的迭代中走到的。

> 这是因为我们每次向左或向右移动**一个**位置，要想「折返」到位置\\(i\\)以及其左侧的位置，我们首先需要在位置\\(i+1\\)向左走到位置\\(i\\)，但这是不可能的。

并且，我们知道位置\\(i+1\\)以及其右侧的位置中一定有一个峰值，因此我们可以设计出如下的一个算法：

- 对于当前可行的下标范围\\([l, r]\\)，我们随机一个下标\\(i\\)；

- 如果下标\\(i\\)是峰值，我们返回\\(i\\)作为答案；

- 如果\\(\textit{nums}[i] < \textit{nums}[i+1]\\)，那么我们抛弃\\([l, i]\\)的范围，在剩余\\([i+1, r]\\)的范围内继续随机选取下标；

- 如果\\(\textit{nums}[i] > \textit{nums}[i+1]\\)，那么我们抛弃\\([i, r]\\)的范围，在剩余\\([l, i-1]\\)的范围内继续随机选取下标。

在上述算法中，如果我们固定选取\\(i\\)为\\([l, r]\\)的中点，那么每次可行的下标范围会减少一半，成为一个类似二分查找的方法，时间复杂度为\\(O(\log n)\\)。

**复杂度分析**

- 时间复杂度：\\(O(\log n)\\)，其中\\(n\\)是数组\\(\textit{nums}\\)的长度。

- 空间复杂度：\\(O(1)\\)。

```cpp
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        int n = nums.size();

        // 辅助函数，输入下标 i，返回一个二元组 (0/1, nums[i])
        // 方便处理 nums[-1] 以及 nums[n] 的边界情况
        auto get = [&](int i) -> pair<int, int> {
            if (i == -1 || i == n) {
                return {0, 0};
            }
            return {1, nums[i]};
        };

        int left = 0, right = n - 1, ans = -1;
        while (left <= right) {
            int mid = (left + right) / 2;
            if (get(mid - 1) < get(mid) && get(mid) > get(mid + 1)) {
                ans = mid;
                break;
            }
            if (get(mid) < get(mid + 1)) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }
        return ans;
    }
};
```

---
title: "LeetCode 2104. Sum of Subarray Ranges"
date: 2022-03-04T13:09:50+08:00
draft: false
tags: ["leetcode", "中等", "栈", "数组", "单调栈"]
math: true
---

You are given an integer array `nums`. The **range** of a subarray of `nums` is the difference between the largest and smallest element in the subarray.

Return _the **sum of all** subarray ranges of_ `nums`_._

A subarray is a contiguous **non-empty** sequence of elements within an array.

<!--more-->

**Example 1:**

> **Input:** nums = [1,2,3]
>
> **Output:** 4
>
> **Explanation:** The 6 subarrays of nums are the following:
> [1], range = largest - smallest = 1 - 1 = 0
>
> [2], range = 2 - 2 = 0
>
> [3], range = 3 - 3 = 0
>
> [1,2], range = 2 - 1 = 1
>
> [2,3], range = 3 - 2 = 1
>
> [1,2,3], range = 3 - 1 = 2
>
> So the sum of all ranges is 0 + 0 + 0 + 1 + 1 + 2 = 4.

**Example 2:**

> **Input:** nums = [1,3,3]
>
> **Output:** 4
>
> **Explanation:** The 6 subarrays of nums are the following:
>
> [1], range = largest - smallest = 1 - 1 = 0
>
> [3], range = 3 - 3 = 0
>
> [3], range = 3 - 3 = 0
>
> [1,3], range = 3 - 1 = 2
>
> [3,3], range = 3 - 3 = 0
>
> [1,3,3], range = 3 - 1 = 2
>
> So the sum of all ranges is 0 + 0 + 0 + 2 + 0 + 2 = 4.

**Example 3:**

> **Input:** nums = [4,-2,-3,4,1]
>
> **Output:** 59
>
> **Explanation:** The sum of all subarray ranges of nums is 59.

**Constraints:**

- `1 <= nums.length <= 1000`
- \\(-10^9 <= nums[i] <= 10^9\\)

**Follow-up:** Could you find a solution with `O(n)` time complexity?

Can you get the max/min of a certain subarray by using the max/min of a smaller subarray within it?

Notice that the max of the subarray from index i to j is equal to max of (max of the subarray from index i to j-1) and nums[j].

## 题解

#### 方法一：遍历子数组

**思路与算法**

为了方便计算子数组的最大值与最小值，我们首先枚举子数组的左边界 i，然后枚举子数组的右边界 j，且 i≤j。在枚举 j 的过程中我们可以迭代地计算子数组 [i,j] 的最小值 minVal 与最大值 maxVal，然后将范围值 maxVal−minVal 加到总范围和。

**复杂度分析**

- 时间复杂度：\\(O(n^2)\\)，其中 n 为数组的大小。两层循环需要 \\(O(n^2)\\)。

- 空间复杂度：O(1)。

```cpp
class Solution {
public:
    long long subArrayRanges(vector<int>& nums) {
        long long ans = 0;
        for(int i = 0; i + 1 < nums.size(); i++) {
            int minimum = nums[i];
            int maximum = nums[i];
            for(int j = i + 1; j < nums.size(); j++) {
                minimum = min(minimum, nums[j]);
                maximum = max(maximum, nums[j]);
                ans += maximum - minimum;
            }
        }
        return ans;
    }
};
```

#### 方法二：单调栈

**思路与算法**

为了使子数组的最小值或最大值唯一，我们定义如果 nums[i]=nums[j]，那么 nums[i] 与 nums[j] 的逻辑大小由下标 i 与下标 j 的逻辑大小决定，即如果 i<j，那么 nums[i] 逻辑上小于 nums[j]。

根据范围和的定义，可以推出范围和 sum 等于所有子数组的最大值之和 sumMax 减去所有子数组的最小值之和 sumMin。

假设 nums[i] 左侧最近的比它小的数为 nums[j]，右侧最近的比它小的数为 nums[k]，那么所有以 nums[i] 为最小值的子数组数目为 (k−i)×(i−j)。为了能获得 nums[i] 左侧和右侧最近的比它小的数的下标，我们可以使用单调递增栈分别预处理出数组 minLeft 和 minRight，其中 minLeft[i] 表示 nums[i] 左侧最近的比它小的数的下标，minRight[i] 表示 nums[i] 右侧最近的比它小的数的下标。

> 以求解 minLeft 为例，我们从左到右遍历整个数组 nums。处理到 nums[i] 时，我们执行出栈操作直到栈为空或者 nums 中以栈顶元素为下标的数逻辑上小于 nums[i]。如果栈为空，那么 minLeft[i]=−1，否则 minLeft[i] 等于栈顶元素，然后将下标 i 入栈。

那么所有子数组的最小值之和 \\(\textit{sumMin} = \sum\_{i=0}^{n-1} (\textit{minRight}[i] - i) \times (i - \textit{minLeft}[i]) \times \textit{nums}[i]\\)。

同理我们也可以求得所有子数组的最大值之和 sumMax。

**复杂度分析**

- 时间复杂度：O(n)，其中 n 为数组的大小。使用单调栈预处理出四个数组需要 O(n)，计算最大值之和与最小值之和需要 O(n)。

- 空间复杂度：O(n)。保存四个数组需要 O(n)；单调栈最多保存 n 个元素，需要 O(n)。

```cpp
class Solution {
public:
    long long subArrayRanges(vector<int>& nums) {
        int n = nums.size();
        vector<int> minLeft(n), minRight(n), maxLeft(n), maxRight(n);
        stack<int> minStack, maxStack;
        for (int i = 0; i < n; i++) {
            while (!minStack.empty() && nums[minStack.top()] > nums[i]) {
                minStack.pop();
            }
            minLeft[i] = minStack.empty() ? -1 : minStack.top();
            minStack.push(i);

            // 如果 nums[maxStack.top()] == nums[i], 那么根据定义，
            // nums[maxStack.top()] 逻辑上小于 nums[i]，因为 maxStack.top() < i
            while (!maxStack.empty() && nums[maxStack.top()] <= nums[i]) {
                maxStack.pop();
            }
            maxLeft[i] = maxStack.empty() ? -1 : maxStack.top();
            maxStack.push(i);
        }
        minStack = stack<int>();
        maxStack = stack<int>();
        for (int i = n - 1; i >= 0; i--) {
            // 如果 nums[minStack.top()] == nums[i], 那么根据定义，
            // nums[minStack.top()] 逻辑上大于 nums[i]，因为 minStack.top() > i
            while (!minStack.empty() && nums[minStack.top()] >= nums[i]) {
                minStack.pop();
            }
            minRight[i] = minStack.empty() ? n : minStack.top();
            minStack.push(i);

            while (!maxStack.empty() && nums[maxStack.top()] < nums[i]) {
                maxStack.pop();
            }
            maxRight[i] = maxStack.empty() ? n : maxStack.top();
            maxStack.push(i);
        }

        long long sumMax = 0, sumMin = 0;
        for (int i = 0; i < n; i++) {
            sumMax += static_cast<long long>(maxRight[i] - i) * (i - maxLeft[i]) * nums[i];
            sumMin += static_cast<long long>(minRight[i] - i) * (i - minLeft[i]) * nums[i];
        }
        return sumMax - sumMin;
    }
};
```

---
title: search-insert-position
tags: [Import-7704]
created: '2021-12-18T11:18:20.412Z'
modified: '2021-12-30T13:58:04.180Z'
---

---
title: "LeetCode 35. 搜索插入位置"
date: 2021-12-08T09:29:58+08:00
draft: false
tags: ["leetcode", "简单", "数组", "二分查找"]
math: true
---

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

<!--more-->

**示例 1:**

> **输入:** nums = [1,3,5,6], target = 5
> 
> **输出:** 2

**示例  2:**

> **输入:** nums = [1,3,5,6], target = 2
> 
> **输出:** 1

**示例 3:**

> **输入:** nums = [1,3,5,6], target = 7
> 
> **输出:** 4

**示例 4:**

> **输入:** nums = [1,3,5,6], target = 0
> 
> **输出:** 0

**示例 5:**

> **输入:** nums = [1], target = 0
> 
> **输出:** 0

**提示:**

- \\(1 <= nums.length <= 10^4\\)
- \\(-10^4 <= nums[i] <= 10^4\\)
- `nums` 为**无重复元素**的**升序**排列数组
- \\(-10^4 <= target <= 10^4\\)

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int l = 0;
        int r = nums.length - 1;
        while (l != r) {
            if (l + 1 == r) {
                if (nums[l] == target) {
                    return l;
                } else if (nums[r] == target) {
                    return r;
                } else if (target > nums[r]) {
                    return r + 1;
                } else if (target < nums[l]) {
                    return l;
                } else {
                    return l + 1;
                }
            }
            int cursor = (l + r) / 2;
            int val = nums[cursor];
            if (val == target) {
                return cursor;
            } else if (val > target) {
                r = cursor;
            } else {
                l = cursor;
            }
        }
        if (nums[l] == target) {
            return l;
        } else if (target > nums[r]) {
            return r + 1;
        } else if (target < nums[l]) {
            return l;
        } else {
            return l + 1;
        }
    }
}
```

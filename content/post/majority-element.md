---
title: "LeetCode 169. 多数元素"
date: 2021-12-25T09:58:05+08:00
draft: false
tags: ["leetcode", "简单", "数组", "哈希表", "分治", "计数", "排序"]
math: true
---

给定一个大小为 _n_ 的数组，找到其中的多数元素。多数元素是指在数组中出现次数 **大于** `⌊ n/2 ⌋`  的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

<!--more-->

**示例  1：**

**输入：** [3,2,3]
**输出：** 3

**示例  2：**

**输入：** [2,2,1,1,1,2,2]
**输出：** 2

**进阶：**

- 尝试设计时间复杂度为 O(n)、空间复杂度为 O(1) 的算法解决此问题。

```cpp
class Solution {
private:
    std::map<int, int> map;
public:
    Solution() {
        this->map = std::map<int, int>();
    }

    int majorityElement(vector<int> &nums) {
        for (int i: nums) {
            map[i] = map[i] + 1;
            if (map[i] > nums.size() / 2) {
                return i;
            }
        }
        return 0;
    }
};
```

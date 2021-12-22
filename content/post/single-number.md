---
title: "LeetCode 136. 只出现一次的数字"
date: 2021-12-21T09:22:19+08:00
draft: false
tags: ["leetcode", "简单", "位运算","数组"]
math: true
---

给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

<!--more-->

**说明：**

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

**示例 1:**

> **输入:** [2,2,1]
> 
> **输出:** 1

**示例  2:**

> **输入:** [4,1,2,1,2]
> 
> **输出:** 4

```java
class Solution {
    public int singleNumber(int[] nums) {
        Set<Integer> setSingle = new HashSet<>();
        Set<Integer> setMulti = new HashSet<>();
        for (int num : nums) {
            if (setMulti.contains(num)) {
                continue;
            }
            if (setSingle.contains(num)) {
                setSingle.remove(num);
                setMulti.add(num);
                continue;
            }
            setSingle.add(num);
        }
        return setSingle.iterator().next();
    }
}
```

---
title: "LeetCode 5965. 相同元素的间隔之和"
date: 2021-12-26T15:11:10+08:00
draft: false
tags: ["leetcode", "中等"]
math: true
---

给你一个下标从 **0** 开始、由 `n` 个整数组成的数组 `arr` 。

`arr` 中两个元素的 **间隔** 定义为它们下标之间的 **绝对差** 。更正式地，`arr[i]` 和 `arr[j]` 之间的间隔是 `|i - j|` 。

返回一个长度为 `n` 的数组  `intervals` ，其中 `intervals[i]` 是 `arr[i]` 和 `arr` 中每个相同元素（与 `arr[i]` 的值相同）的 **间隔之和** _。_

<!--more-->

**注意：**`|x|` 是 `x` 的绝对值。

**示例 1：**

**输入：** arr = [2,1,3,1,2,3,3]
**输出：** [4,2,7,2,4,4,5]
**解释：**

- 下标 0 ：另一个 2 在下标 4 ，|0 - 4| = 4
- 下标 1 ：另一个 1 在下标 3 ，|1 - 3| = 2
- 下标 2 ：另两个 3 在下标 5 和 6 ，|2 - 5| + |2 - 6| = 7
- 下标 3 ：另一个 1 在下标 1 ，|3 - 1| = 2
- 下标 4 ：另一个 2 在下标 0 ，|4 - 0| = 4
- 下标 5 ：另两个 3 在下标 2 和 6 ，|5 - 2| + |5 - 6| = 4
- 下标 6 ：另两个 3 在下标 2 和 5 ，|6 - 2| + |6 - 5| = 5

**示例 2：**

**输入：** arr = [10,5,10,10]
**输出：** [5,0,3,4]
**解释：**

- 下标 0 ：另两个 10 在下标 2 和 3 ，|0 - 2| + |0 - 3| = 5
- 下标 1 ：只有这一个 5 在数组中，所以到相同元素的间隔之和是 0
- 下标 2 ：另两个 10 在下标 0 和 3 ，|2 - 0| + |2 - 3| = 3
- 下标 3 ：另两个 10 在下标 0 和 2 ，|3 - 0| + |3 - 2| = 4

**提示：**

- `n == arr.length`
- \\(1 <= n <= 10^5\\)
- \\(1 <= arr[i] <= 10^5\\)
- 对于在数组中发现的每个独特的值，存储一个排序的数组中具有该值的元素的索引列表。
- 一种方法是使用一个HashMap，将值映射到它们的索引列表。当你遍历数组时，更新这个映射。
- 分别处理每一个索引列表，通过利用前缀和得到该值的元素的间隔之和。
- 对于每个元素，跟踪之前和之后的相同元素的指数之和。利用这一点来计算该元素与其他相同值的元素的间隔之和。

```java
class Solution {
    public long[] getDistances(int[] arr) {
        long[] ans = new long[arr.length];
        HashMap<Integer, LinkedList<Integer>> map = new HashMap<>();
        for (int i = 0; i < arr.length; i++) {
            if (!map.containsKey(arr[i])) {
                LinkedList<Integer> list = new LinkedList<>();
                list.add(i);
                map.put(arr[i], list);
            } else {
                LinkedList<Integer> list = map.get(arr[i]);
                for (int j : list) {
                    ans[j] += Math.abs(j - i);
                    ans[i] += Math.abs(j - i);
                }
                list.add(i);
            }
        }
        return ans;
    }
}
```

---
title: remove-duplicates-from-sorted-array
tags: [Import-7704]
created: '2021-12-18T11:18:20.412Z'
modified: '2021-12-30T13:58:03.565Z'
---

---
title: "LeetCode 26. 删除有序数组中的重复项"
date: 2021-12-06T10:37:06+08:00
draft: false
tags: ["leetcode", "简单", "数组", "双指针"]
math: true
---

给你一个有序数组 `nums` ，请你 **[原地](http://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 **[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** **修改输入数组** 并在使用 O(1) 额外空间的条件下完成。

<!--more-->

**说明:**

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以 **「引用」** 方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```java
// **nums** 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);
// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 **该长度范围内** 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

**示例 1：**

> **输入：** nums = [1,1,2]
> 
> **输出：** 2, nums = [1,2]
> 
> **解释：** 函数应该返回新的长度 **`2`** ，并且原数组 _nums_ 的前两个元素被修改为 **`1`**, **`2`** `。`不需要考虑数组中超出新长度后面的元素。

**示例 2：**

> **输入：** nums = [0,0,1,1,1,2,2,3,3,4]
> 
> **输出：** 5, nums = [0,1,2,3,4]
> 
> **解释：** 函数应该返回新的长度 **`5`** ， 并且原数组 _nums_ 的前五个元素被修改为 **`0`**, **`1`**, **`2`**, **`3`**, **`4`** 。不需要考虑数组中超出新长度后面的元素。

**提示：**

- \\(0 <= nums.length <= 3 \* 10^4\\)

- \\(-10^4 <= nums[i] <= 104\\)

- `nums` 已按升序排列

- 在这个问题中，需要关注的关键点是被排序的输入数组。就重复的元素而言，当给定的数组被排序后，它们在数组中的位置是什么？看看上面的图片就知道答案了。如果我们知道其中一个元素的位置，我们是否也知道所有重复元素的位置？
  
  ![](https://tategotoazarasi.github.io/images/hint_rem_dup.png)

- 我们需要就地修改数组，最终数组的大小有可能小于输入数组的大小。所以，我们应该在这里使用一个双指针的方法。一个是追踪原始数组中的当前元素，另一个是追踪唯一的元素。

- 从本质上讲，一旦遇到一个元素，你只需要绕过它的重复部分，然后转到下一个独特的元素。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length <= 1) {
            return nums.length;
        }
        int ret = nums.length;
        for (int i = 0; i < ret; i++) {
            int j = i + 1;
            while (j < ret) {
                if (nums[i] == nums[j]) {
                    j++;
                } else {
                    break;
                }
            }
            for (int k = 0; i + k + 1 < nums.length; k++) {
                nums[i + k + 1] = nums[Math.min(j + k, nums.length - 1)];
            }
            ret -= j-i-1;
        }
        return ret;
    }
}
```

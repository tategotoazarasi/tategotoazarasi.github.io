---
title: convert-sorted-array-to-binary-search-tree
tags: [Import-7704]
created: '2021-12-18T11:18:20.411Z'
modified: '2021-12-30T13:58:00.552Z'
---

---
title: "LeetCode 108. 将有序数组转换为二叉搜索树"
date: 2021-12-15T10:22:18+08:00
draft: false
tags: ["leetcode", "简单", "树", "二叉搜索树", "数组", "分治", "二叉树"]
math: true
---

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵 **高度平衡** 二叉搜索树。

**高度平衡** 二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过 1 」的二叉树。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/btree1.jpg)

> **输入：** nums = [-10,-3,0,5,9]
>
> **输出：** [0,-3,9,-10,null,5]
>
> **解释：** [0,-10,5,null,-3,null,9] 也将被视为正确答案：
>
> ![](https://tategotoazarasi.github.io/images/btree2.jpg)

**示例 2：**

![](https://tategotoazarasi.github.io/images/btree.jpg)

> **输入：** nums = [1,3]
>
> **输出：** [3,1]
>
> **解释：** [1,3] 和 [3,1] 都是高度平衡二叉搜索树。

**提示：**

- \\(1 <= nums.length <= 10^4\\)
- \\(-10^4 <= nums[i] <= 10^4\\)
- `nums` 按 **严格递增** 顺序排列

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums.length == 1) {
            return new TreeNode(nums[0]);
        }
        if (nums.length == 2) {
            return new TreeNode(nums[0], null, new TreeNode(nums[1]));
        }
        if (nums.length == 3) {
            return new TreeNode(nums[1], new TreeNode(nums[0]), new TreeNode(nums[2]));
        }
        return new TreeNode(nums[nums.length / 2], sortedArrayToBST(Arrays.copyOfRange(nums, 0, nums.length / 2)), sortedArrayToBST(Arrays.copyOfRange(nums, nums.length / 2 + 1, nums.length)));
    }
}
```

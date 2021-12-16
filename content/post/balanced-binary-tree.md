---
title: "LeetCode 110. 平衡二叉树"
date: 2021-12-16T10:02:54+08:00
draft: false
tags: ["leetcode", "简单", "树", "深度优先搜索", "二叉树"]
math: true
---

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

> 一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1 。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/balance_1.jpg)

> **输入：** root = [3,9,20,null,null,15,7]
> 
> **输出：** true

**示例 2：**

![](https://tategotoazarasi.github.io/images/balance_2.jpg)

> **输入：** root = [1,2,2,3,3,null,null,4,4]
> 
> **输出：** false

**示例 3：**

> **输入：** root = []
> 
> **输出：** true

**提示：**

- 树中的节点数在范围 `[0, 5000]` 内
- \\(-10^4 <= Node.val <= 10^4\\)

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        }
        int ldepth = 0;
        int rdepth = 0;
        try {
            if (root.left != null) {
                ldepth = getDepth(root.left);
            }
            if (root.right != null) {
                rdepth = getDepth(root.right);
            }
        } catch (NotBalancedBinaryTreeException e) {
            return false;
        }
        return Math.abs(ldepth - rdepth) <= 1;
    }

    private int getDepth(TreeNode node) throws NotBalancedBinaryTreeException {
        int depth = 0;
        int ldepth = 0;
        int rdepth = 0;
        if (node.left != null) {
            ldepth = getDepth(node.left);
        }
        if (node.right != null) {
            rdepth = getDepth(node.right);
        }
        if (Math.abs(ldepth - rdepth) > 1) {
            throw new NotBalancedBinaryTreeException();
        }
        return Math.max(ldepth, rdepth) + 1;
    }

    private static class NotBalancedBinaryTreeException extends Exception {

    }
}
```
